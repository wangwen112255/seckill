<template>
    <div>
        <el-row>
            <el-col :span="3">
                <el-input v-model="search_key" placeholder="搜索(回车执行)" @blur="search" @keyup.enter.native="search"></el-input>
            </el-col>
        </el-row>
        <el-row>
            <el-table :data="table_data" style="width: 100%; height: 100%;" size="mini" @sort-change="change_sort">
                <el-table-column prop="id" label="ID" width="70" sortable></el-table-column>
                <el-table-column prop="goods_id" label="商品ID" width="100"></el-table-column>
                <el-table-column prop="shopper_id" label="商家名" width="100"></el-table-column>
                <el-table-column prop="title" label="商家名" width="100"></el-table-column>
                <el-table-column prop="pictures" width="120" label="图片">
                    <template slot-scope="scope">
                        <img :src="parse_pickture(scope.row.pictures)" :alt="scope.row.title" style="width: 100px; height: 100px;">
                    </template>
                </el-table-column>
                <el-table-column prop="description" label="商品描述" width="100"></el-table-column>
                <el-table-column prop="begin_time" width="100" label="开始时间"></el-table-column>
                <el-table-column prop="end_time" width="100" label="结束时间"></el-table-column>
                <el-table-column prop="ori_price" width="*" label="商品原价"></el-table-column>
                <el-table-column prop="kill_price" width="200" label="秒杀价格"></el-table-column>
                <el-table-column width="300" label="操作">
                    <template slot-scope="scope" >
                        <div v-if="scope.row.state != 1">
                            <el-button size="small" type="danger" @click="check_seckill(scope.row, 1)">通过</el-button>
                            <el-button size="small" type="danger" @click="check_seckill(scope.row, -1)">不通过</el-button>
                            <el-button size="small" type="danger" @click="delete_seckill(scope.row.id)">删除</el-button>
                        </div>
                        <div v-else>
                            已通过验证
                        </div>
                    </template>
                </el-table-column>
            </el-table>
        </el-row>
        <el-row>
            <el-pagination
                :current-page.sync="page.page_index"
                :page-size="page.per_page"
                @size-change="size_change"
                :page-sizes="[3, 5, 10, 20, 30, 40, 50, 100]"
                class="align-center"
                layout="sizes, total, prev, pager, next"
                :total="seckill_list.length">
            </el-pagination>
        </el-row>
    </div>
</template>

<script>
    import Vue from "vue";
    import {mapState} from "vuex";
    import {Row, Col, Table, TableColumn, Button, Dialog, Pagination, Input} from "element-ui";
    Vue.use(Row);
    Vue.use(Col);
    Vue.use(Table);
    Vue.use(TableColumn);
    Vue.use(Button);
    Vue.use(Dialog);
    Vue.use(Input);
    Vue.use(Pagination);

    export default {
        name: "seckill",
        data() {return {
            search_key: ''
        }},
        computed: {
            ...mapState({
                "seckill_list": state=>state.seckill.seckill_list,
                "page": state=>state.seckill.page
            }),
            table_data() {
                return this.seckill_list.slice((this.page.page_index-1)*this.page.per_page, this.page.page_index*this.page.per_page)
            }
        },
        methods: {
            size_change(size) {
                this.$store.commit("seckill/set_page", {
                    per_page: size
                });
            },
            add_seckill() {
                this.$store.commit("seckill/clear_new_seckill");
                this.$store.commit("seckill/set_visible", {key: "add", value: true});
            },
            check_seckill(seckill, state) {
                const _this = this;
                this.$store.dispatch("seckill/check_seckill", {id: seckill.id, new_state: state}).then(()=>{
                    _this.$store.commit('seckill/update_seckill_by_key', {key: "id", key_val: seckill.id, updates: {state: state}});
                    _this.$success("更新成功！");
                }).catch(error=>{
                    _this.$error(error);
                })
            },
            change_sort({column, prop, order}) {
                this.$store.commit("seckill/set_seckill_list", this.tool.sorted_table_data(this.seckill_list, prop, order));
            },
            search() {
                if (!this.search_key) {
                    this.$store.commit("seckill/reset_seckill_list");
                }else {
                    this.$store.commit("seckill/set_seckill_list", this.tool.search_table_data(this.$store.state.seckill.back_data, this.search_key));
                }
            },
            delete_seckill(id) {
                const _this = this;
                this.$store.dispatch("seckill/delete_seckill", id).then(()=>{
                    _this.$store.commit("seckill/delete_seckill_from_list", id);
                    _this.$success("删除成功！");
                }).catch(error=>_this.$error(error));
            },
            parse_pickture(pictures) {
                return JSON.parse(pictures)[0];
            }
        },
        created() {
            const _this = this;
            if (this.seckill_list.length === 0) {
                this.$store.dispatch('seckill/get_seckill_list').then(x=>{
                    _this.$success("成功加载秒杀申请信息！");
                });
            }
        }
    }
</script>

<style scoped>

</style>