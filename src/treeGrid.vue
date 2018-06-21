<template>
    <div class='autoTable'>
        <table class="table table-bordered el-table" id='hl-tree-table'>
            <thead>
            <tr>
                <th v-for="(column,index) in cloneColumns">
                    <label v-if="column.type === 'selection'">
                        <el-checkbox :indeterminate="isIndeterminateAll" :value="isCheckedAll"
                                     @input="handleCheckAll"/>
                    </label>
                    <label v-else>
                        {{ renderHeader(column, index) }}
                    </label>
                </th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="(item,index) in modelItems" :key="`tr-${index}`" v-show="show(item)"
                :class="{'child-tr':item.parent}">
                <td v-for="(column,snum) in columns" :key="`td-${snum}`" :style=tdStyle(column)>
                    <label v-if="column.type === 'selection'">
                        <el-checkbox :indeterminate="item.isChecked === 2" :value="item.isChecked === 1"
                                     @change="handleCheckClick(item)"/>
                    </label>
                    <div v-if="column.type === 'action'">
                        <el-button :type="action.type" size="small" @click="RowClick(item,$event,index,action.text)"
                                   v-for='action in (column.actions)' :key="action.text">{{action.text}}
                        </el-button>
                    </div>
                    <label @click="toggle(item)" v-if="!column.type">
                            <span v-if='snum==iconRow()'>
                                <i :style="{marginLeft: (item.level - 1) * 15 + 'px'}"
                                   v-if="item.children&&item.children.length>0"
                                   class="iconfont"
                                   :class="{'icon-plus':!item.expanded,'icon-remove':item.expanded }"></i>
                                <i v-else :style="{marginLeft: (item.level - 1) * 15 + 'px'}" class="ms-tree-space"></i>
                            </span> {{renderBody(item,column) }}
                    </label>
                </td>
            </tr>
            </tbody>
        </table>
    </div>
</template>
<script>

    import {deepClone} from "./util";
    import {Button, Checkbox} from 'element-ui';

    export default {
        name: 'treeGrid',
        component: {
            Button,
            Checkbox
        },
        props: {
            columns: Array,
            items: {
                type: Array,
                default: function () {
                    return [];
                }
            },
            expandAll: {
                type: Boolean,
                default: false
            }
        },
        data() {
            return {
                modelItems: [], //处理后数据数组
                cloneColumns: [], //处理后的表头数据
                tdsWidth: 0, //td总宽
                timer: false, //控制监听时长
                dataLength: 0, //树形数据长度
            }
        },
        watch: {
            items() {
                if (this.items) {
                    this.modelItems = []
                    this.initData(deepClone(this.items));
                }
            },
            columns: {
                handler() {
                    this.cloneColumns = this.makeColumns();
                },
                deep: true
            }
        },
        computed: {
            isCheckedAll() {
                return Boolean(this.modelItems.length) && this.modelItems.every(({isChecked}) => isChecked === 1)
            },
            isIndeterminateAll() {
                return this.modelItems.some(({isChecked}) => isChecked === 0) && this.modelItems.some(({isChecked}) => isChecked !== 0)
            }
        },
        mounted() {
            if (this.items) {
                this.modelItems = [];
                this.initData(deepClone(this.items));
                this.cloneColumns = this.makeColumns();
            }
        },
        methods: {
            // 有无多选框折叠位置优化
            iconRow() {
                for (var i = 0, len = this.columns.length; i < len; i++) {
                    if (this.columns[i].type == 'selection') {
                        return 1
                    }
                }
                return 0
            },
            // 设置td宽度,td的align
            tdStyle(column) {
                var style = {}
                if (column.align) {
                    style["text-align"] = column.align;
                }
                if (column.width) {
                    style["min-width"] = column.width + 'px';
                }
                return style;
            },

            // 排序事件
            handleSort(index, type) {
                this.cloneColumns.forEach((col) => col._sortType = 'normal');
                if (this.cloneColumns[index]._sortType === type) {
                    this.cloneColumns[index]._sortType = 'normal'
                } else {
                    this.cloneColumns[index]._sortType = type
                }
                this.$emit('on-sort-change', this.cloneColumns[index]['key'], this.cloneColumns[index]['_sortType'])
            },
            // 点击某一行事件
            RowClick(data, event, index, text) {
                let result = this.makeData(data)
                this.$emit('on-row-click', result, event, index, text)
            },
            // 点击事件 返回数据处理
            makeData(data) {
                const t = this.type(data);
                let o;
                if (t === 'array') {
                    o = [];
                } else if (t === 'object') {
                    o = {};
                } else {
                    return data;
                }

                if (t === 'array') {
                    for (let i = 0; i < data.length; i++) {
                        o.push(this.makeData(data[i]));
                    }
                } else if (t === 'object') {
                    for (let i in data) {
                        if (i != 'spaceHtml' && i != 'parent' && i != 'level' && i != 'expanded' && i != 'isShow') {
                            o[i] = this.makeData(data[i]);
                        }
                    }
                }
                return o;
            },
            // 处理表头数据
            makeColumns() {
                let columns = deepClone(this.columns);
                this.tdsWidth = 0
                columns.forEach((column, index) => {
                    column._index = index;
                    column._width = column.width ? column.width : '';
                    column._sortType = 'normal';
                    this.tdsWidth += column.width ? parseFloat(column.width) : 0;
                });
                return columns;
            },
            // 数据处理 增加自定义属性监听
            initData(data, level = 1, parent = null) {
                if (data instanceof Array) {
                    data.forEach((item) => {
                        this.initData(item, level, parent);
                    })
                } else {
                    data.parent = parent;
                    data.level = level;

                    if ((typeof data.expanded) === "undefined") {
                        data.expanded = this.expandAll;
                    }
                    if (data.children === undefined || data.children.length === 0) {
                        data.expanded = false;
                    }
                    if ((typeof data.show) === "undefined") {
                        data.isShow = this.expandAll;
                    }
                    if ((typeof data.isChecked) === "undefined") {
                        data.isChecked = 0;
                    }
                    this.modelItems.push(data);

                    if (data.children.length) {
                        let checkedAll = true;
                        let hasChecked = false;
                        data.children.forEach((item) => {
                            if (item.isChecked === 0) {
                                checkedAll = false
                            } else {
                                hasChecked = true
                            }
                        });
                        if (checkedAll) {
                            data.isChecked = 1;
                        } else if (!hasChecked) {
                            data.isChecked = 0
                        } else {
                            data.isChecked = 2
                        }
                        this.initData(data.children, level + 1, data);
                    }
                }
            },
            //  隐藏显示
            show(item) {
                return ((item.level === 1) || (item.parent && this.findNodeLike(item.parent).expanded && item.isShow));
            },
            toggle(item) {
                const expanded = item.expanded;
                item.expanded = !expanded;
                if (item.children) {
                    if (expanded) {
                        this.close(item.children);
                    } else {
                        this.open(item.children);
                    }
                }
                this.$forceUpdate()
            },
            findNodeLike(node) {
                let nodeLike = node;
                const items = this.modelItems;
                for (let i = 0, length = items.length; i < length; i++) {
                    if (node.id === items[i].id) {
                        nodeLike = items[i]
                    }
                }
                return nodeLike;
            },
            findIndex(node) {
                return this.modelItems.findIndex(({id}) => id === node.id);
            },
            open(items) {
                items.forEach((child) => {
                    this.$set(this.modelItems[this.findIndex(child)], 'isShow', true);
                    if (child.children && child.expanded) {
                        this.open(child.children);
                    }
                })
            },
            close(items) {
                items.forEach((child) => {
                    this.$set(this.modelItems[this.findIndex(child)], 'isShow', false);
                    if (child.children) {
                        this.close(child.children);
                    }
                })
            },
            //点击check勾选框,判断是否有children节点 如果有就一并勾选
            handleCheckClick(data) {
                this.setNode(data, data.isChecked !== 0 ? 0 : 1);
                this.setParent(data);
                this.$emit('on-selection-change', deepClone(this.modelItems).filter(({isChecked}) => isChecked !== 0));
            },
            setNode(data, checked) {
                if (data instanceof Array) {
                    data.forEach((item) => {
                        this.setNode(item, checked)
                    })
                } else {
                    this.$set(this.modelItems[this.findIndex(data)], 'isChecked', checked);
                    if (data.children && data.children.length) {
                        this.setNode(data.children, checked);
                    }
                }
            },
            setParent(data, isIndeterminate = false) {
                if (data.parent) {
                    let isChecked = 0;
                    if (isIndeterminate) {
                        isChecked = 2;
                    } else {
                        const siblings = this.modelItems.filter(({parent}) => (parent && parent.id) === data.parent.id);
                        let checkAll = true;
                        let hasChecked = false;
                        siblings.forEach(({isChecked}) => {
                            if (isChecked === 0) {
                                checkAll = false;
                            } else {
                                hasChecked = true;
                            }
                        });
                        if (checkAll) {
                            isChecked = 1;
                        } else if (!hasChecked) {
                            isChecked = 0
                        } else {
                            isChecked = 2
                        }
                    }
                    this.$set(this.modelItems[this.findIndex(data.parent)], 'isChecked', isChecked)
                    if (data.parent.parent) {
                        this.setParent(data.parent, isChecked === 2);
                    }
                }
            },
            //checkbox 全选 选择事件
            handleCheckAll(checked) {
                this.modelItems = this.modelItems.map((item) => {
                    const _item = deepClone(item);
                    _item.isChecked = checked ? 1 : 0;
                    return _item;
                })
                this.$emit('on-selection-change', deepClone(this.modelItems).filter(({isChecked}) => isChecked !== 0));
            },
            // 返回表头
            renderHeader(column, $index) {
                if ('renderHeader' in this.columns[$index]) {
                    return this.columns[$index].renderHeader(column, $index);
                } else {
                    return column.title || '#';
                }
            },

            // 返回内容
            renderBody(row, column, index) {
                return row[column.key]
            },
            type(obj) {
                var toString = Object.prototype.toString;
                var map = {
                    '[object Boolean]': 'boolean',
                    '[object Number]': 'number',
                    '[object String]': 'string',
                    '[object Function]': 'function',
                    '[object Array]': 'array',
                    '[object Date]': 'date',
                    '[object RegExp]': 'regExp',
                    '[object Undefined]': 'undefined',
                    '[object Null]': 'null',
                    '[object Object]': 'object'
                };
                return map[toString.call(obj)];
            }
        }
    }
</script>
<style>
    .autoTable {
        overflow: auto;
        width: 100%;
    }

    table {
        width: 100%;
        border-spacing: 0;
        border-collapse: collapse;
    }

    .table-bordered {
        border: 1px solid #EBEBEB;
    }

    .table > tbody > tr > td,
    .table > tbody > tr > th,
    .table > thead > tr > td,
    .table > thead > tr > th {
        border-top: 1px solid #e7eaec;
        line-height: 1.42857;
        padding: 8px;
        vertical-align: middle;
    }

    .table-bordered > tbody > tr > td,
    .table-bordered > tbody > tr > th,
    .table-bordered > tfoot > tr > td,
    .table-bordered > tfoot > tr > th,
    .table-bordered > thead > tr > td,
    .table-bordered > thead > tr > th {
        border: 1px solid #e7e7e7;
    }

    .table > thead > tr > th {
        border-bottom: 1px solid #DDD;
    }

    .table-bordered > thead > tr > td,
    .table-bordered > thead > tr > th {
        background-color: #F5F5F6;
    }

    #hl-tree-table > tbody > tr {
        background-color: #fbfbfb;
    }

    #hl-tree-table > tbody > .child-tr {
        background-color: #fff;
    }

    .ms-tree-space {
        position: relative;
        top: 1px;
        display: inline-block;
        font-style: normal;
        font-weight: 400;
        line-height: 1;
        width: 14px;
        height: 14px;
    }

    .ms-tree-space::before {
        content: ""
    }

    #hl-tree-table th > label {
        margin: 0;
    }
</style>