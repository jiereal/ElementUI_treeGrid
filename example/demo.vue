<template>
    <tree-grid :columns="columns"
               :items="items"
               :expandAll="true"
               @on-row-click="click"
               @on-selection-change="select"
    />
</template>
<script>
    import treeGrid from '../src/treeGrid';

    export default {
        data() {
            return {
                columns: [{
                    type: 'selection',
                    width: '50',
                    align: 'center'
                }, {
                    title: '编码',
                    key: 'code',
                    sortable: true,
                    width: '150',
                }, {
                    title: '名称',
                    key: 'name',
                    width: '150',
                }, {
                    title: '类型',
                    key: 'type',
                    width: '150',

                }, {
                    title: '描述',
                    key: 'description',
                    width: '150',
                }],
                items: [{
                    code: '1',
                    name: 'name1',
                    type: 'type1',
                    description: 'description',
                    id: 1,
                    children: [{
                        code: '1',
                        name: 'name1',
                        type: 'type1',
                        description: 'description',
                        id: 11,
                    }, {
                        code: '1',
                        name: 'name1',
                        type: 'type1',
                        description: 'description',
                        id: 12
                    }, {
                        code: '1',
                        name: 'name1',
                        type: 'type1',
                        description: 'description',
                        id: 13
                    }]
                }, {
                    code: '1',
                    name: 'name1',
                    type: 'type1',
                    description: 'description',
                    id: 2
                }, {
                    code: '1',
                    name: 'name1',
                    type: 'type1',
                    description: 'description',
                    id: 3
                }, {
                    code: '1',
                    name: 'name1',
                    type: 'type1',
                    description: 'description',
                    id: 4
                }]
            }
        },
        methods: {
            click(result, event, index, text) {
                console.log(result, event, index, text)
            },
            select(items) {
                console.log(items)
            }
        },
        components: {
            treeGrid
        }
    }
</script>