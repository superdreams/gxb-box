<style scoped>
    .ivu-input-number {
        width: 100%;
    }

    h3 {
        padding-top: 10px;
        font-weight: normal;
    }

    .split-line {
        height: 1px;
        background: #eee;
        margin: 20px 0;
    }

    .setting-btn {
        text-align: center;
    }
</style>
<template>
    <div class="setting-api">
        <div class="setting-header">
            <h2>接入点</h2>
            <h3>修改API、访问地址、启动端口及水龙头服务器</h3>
        </div>
        <div class="split-line"></div>
        <div class="setting-cont">
            <Form ref="formValidate" :model="formValidate" :rules="ruleValidate" :label-width="100">
                <FormItem label="盒子访问地址" prop="box_url">
                    <i-input v-model="formValidate.box_url" placeholder="请输入访问地址"></i-input>
                </FormItem>
                <FormItem label="盒子启动端口" prop="port">
                    <InputNumber v-model="formValidate.port" placeholder="请输入端口"></InputNumber>
                </FormItem>
                <FormItem label="水龙头地址" prop="faucet_url">
                    <i-input v-model="formValidate.faucet_url" placeholder="请输入水龙头地址"></i-input>
                </FormItem>
                <Collapse value="api_list" style="margin-bottom: 20px">
                    <Panel name="api_list">
                        API服务器
                        <div slot="content">
                            <p v-for="(item,index) in api_list" :value="item" :key="index">{{ item }}</p>
                        </div>
                    </Panel>
                </Collapse>
            </Form>
        </div>
        <div class="setting-btn">
            <Button type="primary" @click="saveConfig()">保存配置</Button>
            <Button type="success" @click="addApiServer()">添加API服务器</Button>
            <Button type="error" @click="removeApiServer()">移除API服务器</Button>
        </div>

        <Modal v-model="add_modal" title="添加新的 websocket API" @on-ok="handleAdd()">
            <i-input v-model="add_api">
                <span slot="prepend">wss://</span>
            </i-input>
        </Modal>

        <Modal v-model="remove_modal" title="删除 websocket API" @on-ok="handleRemove()">
            <Select v-model="remove_api">
                <Option v-for="(item,index) in api_list" :value="item" :key="index">{{ item }}</Option>
            </Select>
        </Modal>
    </div>
</template>
<script>
    import Handler from '../../libs/handler';
    import {mapGetters, mapActions} from 'vuex';

    export default {
        data () {
            return {
                formValidate: {
                    port: 3000,
                    box_url: '',
                    faucet_url: ''
                },
                ruleValidate: {
                    box_url: [
                        {required: true, type: 'string', message: '请输入盒子访问地址', trigger: 'blur'}
                    ],
                    port: [
                        {required: true, type: 'integer', message: '只能输入数字', trigger: 'blur'}
                    ],
                    faucet_url: [
                        {required: true, type: 'url', message: '水龙头地址必须为URL', trigger: 'blur'}
                    ]
                },
                api_list: [],
                add_modal: false,
                remove_modal: false,
                add_api: '',
                remove_api: ''
            };
        },
        created () {
            this.$http.get('/api/get_ip_address').then((res) => {
                this.api_list = this.config.common.witnesses;
                this.formValidate.port = Number(this.config.common.port);
                this.formValidate.faucet_url = this.config.common.faucet_url;
                this.formValidate.box_url = localStorage.getItem('__gxbBox__ApiServer') ? localStorage.getItem('__gxbBox__ApiServer') : 'http://' + res.data;
            }).catch((err) => {
                Handler.error(err);
            });
        },
        computed: {
            ...mapGetters({
                config: 'config'
            })
        },
        methods: {
            ...mapActions({
                setConfig: 'setConfig'
            }),
            handleAdd () {
                if (this.add_api === '') {
                    this.$Message.error('地址不能为空');
                } else {
                    this.api_list.push('wss://' + this.add_api);
                }
            },
            handleRemove () {
                if (this.remove_api === '') {
                    this.$Message.error('请选择要删除的地址');
                } else {
                    this.api_list = this.api_list.filter(t => t !== this.remove_api);
                }
            },
            saveConfig () {
                this.$refs['formValidate'].validate((valid) => {
                    if (valid) {
                        if (this.api_list.length > 0) {
                            this.config.common.port = Number(this.formValidate.port);
                            this.config.common.faucet_url = this.formValidate.faucet_url;
                            this.config.common.witnesses = this.api_list;
                            localStorage.setItem('__gxbBox__ApiServer', this.formValidate.box_url);
                            this.$http({
                                method: 'post',
                                url: '/api/save_config',
                                data: { config: this.config }
                            }).then(() => {
                                this.setConfig({config: this.config});
                                this.$Message.success('保存成功');
                                this.$emit('restart');
                            }).catch((err) => {
                                this.$Message.error('保存失败:' + Handler.error(err));
                            });
                        } else {
                            this.$Message.error('API服务器列表不能为空');
                        }
                    } else {
                        this.$Message.error('验证失败');
                    }
                });
            },
            addApiServer () {
                this.add_modal = true;
            },
            removeApiServer () {
                this.remove_modal = true;
            }
        }
    };
</script>
