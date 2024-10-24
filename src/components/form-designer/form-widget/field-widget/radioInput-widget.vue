<template>
    <form-item-wrapper
        :designer="designer"
        :field="field"
        :rules="rules"
        :design-state="designState"
        :parent-widget="parentWidget"
        :parent-list="parentList"
        :index-of-parent-list="indexOfParentList"
        :sub-form-row-index="subFormRowIndex"
        :sub-form-col-index="subFormColIndex"
        :sub-form-row-id="subFormRowId"
    >
        <div v-if="field.options.optionItems.length === 1">
            <span
                v-for="(it, i) in field.options.settingList[0]"
                :key="i"
            >
                <span v-if="it.type == 'input'">
                    <el-input
                        class="custom-input"
                        :style="{ width: `${it.innerWidth * 10}px` }"
                        @input="onInput(field.options.optionItems[0].value)"
                        v-model="
                            formModel[`${field.options.name}#inputValue`][`${field.options.optionItems[0].value}-${it.inputSort}`]
                        "
                    />
                </span>
                <span v-else>{{ it.label }}</span>
            </span>
        </div>
        <el-radio-group
            v-else
            ref="fieldEditor"
            v-model="fieldModel"
            :disabled="field.options.disabled"
            :size="widgetSize"
            @change="handleChangeEvent"
        >
            <el-radio
                v-for="(item, index) in field.options.optionItems"
                :key="index"
                :label="item.value"
                :disabled="item.disabled"
                :border="field.options.border"
                :style="{ display: field.options.displayStyle }"
            >
                <template #default>
                    <span
                        v-for="(it, i) in field.options.settingList[index]"
                        :key="i"
                    >
                        <span v-if="it.type == 'input'">
                            <el-input
                                class="custom-input"
                                :style="{ width: `${it.innerWidth * 10}px` }"
                                v-model="
                                    formModel[`${field.options.name}#inputValue`][`${item.value}-${it.inputSort}`]
                                "
                                @input="fieldModel = item.value, this.formModel[`${this.field.options.name}#inputValue`] = this.cleanObject(item.value, this.formModel[`${this.field.options.name}#inputValue`])"
                            />
                        </span>
                        <span v-else>{{ it.label }}</span>
                    </span>
                </template>
            </el-radio>
        </el-radio-group>
    </form-item-wrapper>
</template>

<script>
import FormItemWrapper from "./form-item-wrapper";
import emitter from "@/utils/emitter";
import i18n, { translate } from "@/utils/i18n";
import fieldMixin from "@/components/form-designer/form-widget/field-widget/fieldMixin";
import formModel from "@/components/form-designer/form-widget/field-widget/fieldMixin";

export default {
    name: "radioInput-widget",
    componentName: "FieldWidget", //必须固定为FieldWidget，用于接收父级组件的broadcast事件
    mixins: [emitter, fieldMixin, i18n, formModel],
    props: {
        field: Object,
        parentWidget: Object,
        parentList: Array,
        indexOfParentList: Number,
        designer: Object,

        designState: {
            type: Boolean,
            default: false,
        },

        subFormRowIndex: {
            /* 子表单组件行索引，从0开始计数 */ type: Number,
            default: -1,
        },
        subFormColIndex: {
            /* 子表单组件列索引，从0开始计数 */ type: Number,
            default: -1,
        },
        subFormRowId: {
            /* 子表单组件行Id，唯一id且不可变 */ type: String,
            default: "",
        },
    },
    components: {
        FormItemWrapper,
    },
    data() {
        return {
            oldFieldValue: null, //field组件change之前的值
            fieldModel: null,
            rules: [],
        };
    },
    computed: {},
    beforeCreate() {
        /* 这里不能访问方法和属性！！ */
    },

    created() {
        /* 注意：子组件mounted在父组件created之后、父组件mounted之前触发，故子组件mounted需要用到的prop
           需要在父组件created中初始化！！ */
        this.initOptionItems();
        this.initFieldModel();
        this.registerToRefList();
        this.initEventHandler();
        this.buildFieldRules();
        if (!this.field.options.settingList?.length) {
            this.buildLabelSetting();
        }
        if(!Object.prototype.hasOwnProperty.call(this.formModel, `${this.field.options.name}#inputValue`)){
            this.formModel[`${this.field.options.name}#inputValue`] = {}
        }
    },
    beforeMount() {},
    beforeUnmount() {
        this.unregisterFromRefList();
    },
    watch: {
        "field.options.optionItems": {
            handler(newVal, oldVal) {
                this.buildLabelSetting();
            },
            deep: true,
        },
        fieldModel: {
            handler(newVal, oldVal) {
                this.formModel[this.field.options.name] = newVal;
            },
            deep: true,
        }
    },
    beforeMount() {
        this.field.options.labelInput = {};
    },
    methods: {
        buildLabelSetting() {
            if (!this.field.options.settingList) {
                this.field.options.settingList = [];
            }
            let inputSort = 0;

            this.field.options?.optionItems?.forEach((item, i) => {
                inputSort = 0;
                this.field.options.settingList[i] = [];
                let matches = item.label.match(/_+/g); //
                let lengths = matches?.map((match) => match.length) || [];
                let str = item.label.replace(/_+/g, "**");
                let arr = str.split("**");
                if (!arr[0]) {
                    this.field.options.settingList[i].push({
                        type: "input",
                        label: "",
                        inputSort,
                        innerWidth: lengths[0] || 10,
                    });
                    inputSort++;
                    arr.shift();
                    lengths.shift();
                }
                arr.forEach((item, index) => {
                    if (index == arr.length - 1) {
                        if (item) {
                            this.field.options.settingList[i].push({
                                type: "string",
                                label: item,
                            });
                        }
                    } else {
                        this.field.options.settingList[i].push({
                            type: "string",
                            label: item,
                        });
                        this.field.options.settingList[i].push({
                            type: "input",
                            label: "",
                            inputSort,
                            innerWidth: lengths[0] || 10,
                        });
                        inputSort++;
                        lengths.shift();
                    }
                });
            });
        },
        onInput(radioValue) {
            if (this.fieldModel != radioValue) {
                this.fieldModel = radioValue
            }
        },
        handleChangeEvent(selectedValue){
            this.formModel[`${this.field.options.name}#inputValue`] = this.cleanObject(selectedValue, this.formModel[`${this.field.options.name}#inputValue`])
        },
        cleanObject(selectedValue, obj) {
            const cleanedObj = { ...obj };
            // 获取所有选中的主键
            const selectedKeys = new Set([selectedValue]);
            // 遍历对象的键
            for (const key in obj) {
                // 获取主键部分
                const mainKey = key.split('-')[0];
                // 如果主键不在选中的值中，删除该键值对
                if (!selectedKeys.has(mainKey)) {
                delete cleanedObj[key];
                }
            }
            return cleanedObj;
        },
        async initOptionItems(){
            // 这里的$GETDICT;$HTTP;$CONFIG来自uni-web项目挂载的全局对象
            let optionItems = this.field.options?.dictName ? this.$GETDICT(this.field.options.dictName) : []
            if(this.field.options?.optionApi) {
            const url = this.field.options.optionApi;
            const res = await this.$HTTP.post(this.$CONFIG.API_URL + url, {});
            optionItems = res.data || []
            }
            this.field.options.optionItems = optionItems
        }
    },
};
</script>

<style lang="scss" scoped>
@import "../../../../styles/global.scss"; /* form-item-wrapper已引入，还需要重复引入吗？ */
.custom-input {
    :deep(.el-input__wrapper) {
        background-color: unset;
        box-shadow: unset !important;
        padding: 0;
        .el-input__inner {
            border-bottom: 1px solid #000;
            line-height: 18px;
            height: 18px;
            text-align: center;
            color: inherit;
        }
    }
}
:deep(.el-radio__input.is-checked + .el-radio__label .custom-input .el-input__wrapper .el-input__inner) {
    border-bottom: 1px solid var(--el-color-primary);
}
</style>
