RemoteSelect.vue
```
<template>
    <div>
        <div class="taginput" v-if="isshowtag">
            <el-tag :type="tagtype" :key="val" :closable="istagclose" @close="tagclose">{{this.vallabel}}</el-tag>
        </div>
        <el-select
            v-if="!isshowtag"
            :disabled="selectdisable"
            filterable
            v-model="val"
            remote
            clearable
            reserve-keyword
            :placeholder="placeholder"
            :remote-method="search"
            @change="onselected"
            @clear="clear"
            @keyup.enter.native="create"
            :loading="loading">
            <el-option
                v-for="(item,index) in datalist"
                :key="index"
                :label="item.label"
                :value="item.value">
            </el-option>
            <el-pagination
                @prev-click="prepage"
                @next-click="nextpage"
                background
                :layout="layout">
            </el-pagination>

        </el-select>

    </div>
</template>
<script>
    export default {
        name: 'remote-select',
        data() {
            return {
                isinit: false,
                istagclose: true,
                selectdisable: false,
                val: '',
                vallabel: 'xxx',
                isfoucs: false,
                keyword: '',
                page: 1,
                layout: 'prev, next',
                dlv: '',
                datalist: [],
                loading: false

            }
        },
        props: {
            searchable: {
                type: String,
                default: 'false'
            },
            readonly: {
                type: String,
                default: 'false'
            },
            tagtype: {
                type: String,
                default: ''
            },
            showtag: {
                default: 'true'
            },
            initv: {
                default: ''
            },
            initlabel: {
                default: ''
            },
            pagesize: {
                type: Number,
                default: 5,

            },
            placeholder: {
                type: String,
                default: 'Search'

            },
            valueKey: {
                default: 'id'
            },
            labelKey: {
                default: '',
                required: true
            },
            url: {
                type: String,
                default: '',
                required: true
            },
            addurl: {
                type: String,
                default: ''
            }

        },

        methods: {
            search(query) {
                /* if (this.searchable == 'true' || this.isinit == true) {
                     if (this.isinit == true) {
                         this.isinit = false
                     }*/
                this.query = query;
                var qo = {};
                qo[this.labelKey] = query
                var params = {
                    page: this.page,
                    size: this.pagesize,
                    query: qo
                }
                params['orderby'] = this.valueKey + ' desc'
                //调用post请求
                this.$axios.post(this.url, params).then(res => {
                    //总数赋值
                    //页面元素赋值
                    if (res.data.rows && res.data.rows.length > 0) {
                        this.datalist = []
                        var rs = [];
                        res.data.rows.forEach((item, idx) => {
                            var o = {};
                            o['value'] = item[this.valueKey]
                            o['label'] = item[this.labelKey]
                            rs.push(o)
                        })

                        if (rs.length < this.pagesize) {
                            this.layout = 'prev'
                        } else {
                            this.layout = 'prev,next'

                        }
                        this.datalist = rs;
                    } else {
                        this.datalist = []
                    }

                }).catch(e => this.pageLoading = false)


                /*    }else{

                        this.datalist = this.list.filter(item => {
                            return item.label.toLowerCase()
                                .indexOf(query.toLowerCase()) > -1;
                        });

                    }*/

            },
            onselected(value) {
                if (value) {
                    this.val = value
                    for (var i = 0; i < this.datalist.length; i++) {
                        if (this.datalist[i].value === value) {
                            this.$emit('input', this.datalist[i].value);
                            this.$emit("change", this.datalist[i])
                            if (this.showtag == true || this.showtag == 'true') {
                                this.isshowtag = true;
                                this.vallabel = this.datalist[i].label
                            }
                        }
                    }
                }

            },
            clear() {
                this.val = ''
                this.vallabel = ''
                this.$emit('input', '');
                this.$emit("change", {value: '', label: ''})

            },
            create() {
                var label = this.$el.querySelector("input").value;
                var params = {}
                params[this.labelKey] = label;
                if (this.addurl && label) {
                    this.$axios.post(this.addurl, params).then(res => {
                        //总数赋值
                        //页面元素赋值

                        if (!res.data.success) {
                            this.$message({
                                type: 'error',
                                message: 'Enter Add Error'
                            })
                            return
                        }

                        var val = res.data.value;
                        this.val = val;
                        this.vallabel = label;
                        this.isshowtag = true;


                    }).catch(e => this.pageLoading = false)
                }


            },
            tagclose() {
                this.val = ''
                this.vallabel = ''
                this.isshowtag = false;

            },
            prepage(cp) {

                if (this.page > 1) {
                    this.page--
                }
                this.search(this.query)
            },
            nextpage(cp) {
                this.page++
                this.search(this.query)
            },
            initdata() {
                if (this.initv && this.initlabel) {
                    var exist = false;
                    this.datalist.forEach((n, index) => {
                        if (n.value == this.initv) {
                            exist = true
                            this.onselected(this.initv)

                        }
                    })
                    if (!exist) {
                        var i = {}
                        i['value'] = this.initv
                        i['label'] = this.initlabel
                        this.datalist.push(i)
                        this.onselected(this.initv)

                    }


                } else {
                    this.$emit('input', null);
                    this.$emit("change", {})
                    if (this.showtag == true || this.showtag == 'true') {
                        if (!this.val) {
                            this.vallabel = ''
                        }

                    }
                }
            }
        },
        watch: {
            readonly: function () {
                if (this.readonly == 'true' || this.readonly == true) {
                    this.selectdisable = true
                    this.istagclose = false
                } else {
                    this.selectdisable = false
                    this.istagclose = true
                }
            },
            initv: function (val) {

                if (!this.initv) {
                    this.val = ''
                    if (this.showtag == true || this.showtag == 'true') {
                        this.isshowtag = false;

                        this.vallabel = ''
                    }
                }

                this.initdata();
            }


        },
        mounted() {
            this.val = ''
            this.isinit = true
            this.search(this.query);
            this.initdata();

            //如果是readonly
            if (this.readonly == 'true') {
                this.selectdisable = true
                this.isshowtag = false
            }
        }
    }

</script>
<style scoped>
    .taginput {
        border: 1px solid #dcdfe6;
        border-radius: 4px;
        min-width: 173px;
        max-width: 200px;
        padding-left: 2px;
        height: 28px;
    }
</style>



```
使用
```
 <remote-select label-key="labelname" showtag="true" 
 readonly="false" v-model="form1.labelid" :initv="form1.labelid"
 :initlabel="form1.labelname" 
 url="dspCampaignsLabel/loadPage" 
 addurl="dspCampaignsLabel/save"
 placeholder="Label"></remote-select>
```
