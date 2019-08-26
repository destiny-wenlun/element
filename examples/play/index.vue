<template>
    <div>
        <div>
            <el-button type="priamry" @click="getExpandedAll">获取展开的节点</el-button>
            <el-button type="priamry" @click="setExpandedAll">展开指定节点</el-button>
        </div>
        <el-tree ref="tree" :data="data" :props="defaultProps" node-key="id"></el-tree>
    </div>
</template>

<script>
export default {
    data() {
        return {
            data: [
                {
                    id: 1,
                    label: "一级 1",
                    children: [
                        {
                            id: 123,
                            label: "二级 1-1",
                            children: [
                                {
                                    id: 124,
                                    label: "三级 1-1"
                                }
                            ]
                        }
                    ]
                },
                {
                    id: 2,
                    label: "一级 2",
                    children: [
                        {
                            id: 3,
                            label: "二级 2-1"
                        },
                        { id: 4, label: "二级 2-2" }
                    ]
                },
                {
                    id: 5,
                    label: "一级 3",
                    children: [
                        {
                            id: 6,
                            label: "二级 3-1"
                        },
                        {
                            id: 7,
                            label: "二级 3-2"
                        }
                    ]
                }
            ],
            defaultProps: {
                children: "children",
                label: "label"
            }
        };
    },
    methods: {
        handleNodeClick(data) {
            console.log(data);
        },
        getExpandedAll() {
            console.log(this.$refs.tree.getExpandedKeys());
        },
        setExpandedAll() {  
            this.$refs.tree.setExpandedKeys([1,123,5]);
        }
    }
};
</script>