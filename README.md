RemoteSelect.vue
```
<template>
    <div>
        <div class="taginput" v-show="isshowtag">
            <el-tag :type="tagtype" :key="val" :closable="istagclose" @close="tagclose">{{vallabel}}</el-tag>
        </div>
        <el-select
            v-show="!isshowtag"
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
                vallabel: '',
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
                default: 'false'
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
                this.val = value

                this.datalist.forEach((item, ids) => {
                    if (item.value == value) {
                        this.val = value
                        this.vallabel = item.label
                        this.$emit('input', item.value);
                        this.$emit("change", item)
                    }
                })
            },
            clear() {
                this.val = ''
                this.vallabel = ''
                this.$emit('input', '');
                this.$emit("change", {value: '', label: ''})

            },
            tagclose() {
                this.val = ''
                this.vallabel = ''
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
                        this.onselected(this.initv)
                        this.datalist.push(i)
                    }

                }
            }
        },
        watch: {
            initv: function (val) {
                this.initdata();
            }

        },
        computed: {
            isshowtag: function () {
                if (this.showtag === 'true' && this.val != '' && this.val) {
                    return true;
                } else {
                    return false;
                }
            }
        },
        mounted() {
            this.isinit = true
            this.search(this.query);
            this.initdata();

            //如果是readonly
            if (this.readonly == 'true') {
                this.selectdisable = true
                this.istagclose = false
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
 :initlabel="form1.labelname" url="dspCampaignsLabel/loadPage"
 placeholder="Label"></remote-select>
```
