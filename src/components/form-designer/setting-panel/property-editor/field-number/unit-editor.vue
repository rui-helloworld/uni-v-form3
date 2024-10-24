<template>
    <el-form-item :label="i18nt('designer.setting.unit')">
        <el-radio-group v-model="optionModel.unit" @change="change">
            <el-radio label="one" size="large">元</el-radio>
            <el-radio label="tenThousand" size="large">万元</el-radio>
            <el-radio label="hundredMillion" size="large">亿元</el-radio>
        </el-radio-group>
    </el-form-item>
</template>

<script>
import i18n from "@/utils/i18n";

export default {
    name: "unit-editor",
    mixins: [i18n],
    props: {
        designer: Object,
        selectedWidget: Object,
        optionModel: Object,
    },
};
</script>

<style scoped></style>
