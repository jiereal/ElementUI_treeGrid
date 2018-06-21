# ElementUI_treeGrid
  treeGrid component with element-ui library

# Installation

```
    npm i elementui_treegrid
```

# Example
```js
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
                        description: 'description'
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
```

# Contribution
If you find a bug or want to contribute to the code or documentation, you can help by submitting an [issue](https://github.com/jiereal/vuex_helper/issues) or a [pull request](https://github.com/jiereal/vuex_helper/pulls) .

# License
This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details