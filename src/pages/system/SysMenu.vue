<template>
    <div class="sysbasic-container">
        <el-tabs v-model="defaultTab" type="menuMana">
            <menu-mana></menu-mana>
        </el-tabs>
    </div>
</template>
<script>

import MenuMana from "./basic/MenuMana"

export default {
    name: "SysBasic",
    data() {
        return {
            defaultTab: 'depMana'
        }
    },
    components: {
        'menu-mana': MenuMana
       
    }
}
</script>
<style lang="stylus" scoped>
    .sysbasic-container
        margin-top 10px
</style>
