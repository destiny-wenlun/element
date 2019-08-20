<template>
    <div style="margin: 20px;">
        <div style="padding:10px;">
            <el-button type="primary">按钮1</el-button>
            <el-button type="info">按钮2</el-button>
            <el-button type="danger" plain>按钮3</el-button>
            <el-button type="warning" plain>按钮4</el-button>
            <el-button>按钮5</el-button>
            <el-button @click="modal=true">打开模态框</el-button>
        </div>
        <el-table :data="data" style="width:900px" show-summary border>
            <el-table-column label="姓名" prop="name" :width="200" fixed="left"></el-table-column>
            <el-table-column label="年龄" prop="age" :width="200"></el-table-column>
            <el-table-column label="性别" prop="sex" :width="200"></el-table-column>
            <el-table-column label="民族" prop="nation" :width="200"></el-table-column>
            <el-table-column label="身份证号" prop="idcard" fixed="right" :width="300"></el-table-column>
        </el-table>
        <el-dialog title="模态框" :visible.sync="modal" appendToBody>
            <div v-for="i in 100" :key="i">
                这是内容吗？
                这是内容吗？
                这是内容吗？
                这是内容吗？
                这是内容吗？
                <el-input />
            </div>
            <template slot="footer">
                <el-button>確定</el-button>
            </template>
        </el-dialog>
    </div>
</template>

<script>
export default {
    data() {
        return {
            data: [],
            modal: false
        };
    },
    mounted() {
        setTimeout(() => {
            for (let i = 0; i < 10; i++) {
                this.data.push({
                    name: `姓名${i}`,
                    age: 20,
                    sex: "男",
                    nation: "汉族",
                    idcard: "522128"
                });
            }
        }, 500);
    }
};
</script>