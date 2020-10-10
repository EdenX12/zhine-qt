<template>
    <div class="sysbasic-container">
        <el-tabs v-model="defaultTab" type="menuMana">
            <menu-role></menu-role>
        </el-tabs>
    </div>
</template>
<script>

import MenuRole from "./basic/MenuRole"

export default {
    name: "SysBasic",
    data() {
        return {
            defaultTab: 'depMana'
        }
    },
    components: {
        'menu-role': MenuRole
    }
}
</script>
<style lang="stylus" scoped>
    .sysbasic-container
        margin-top 10px
</style>
