<template>
    <div class="board">
        <v-flex xs12 sm12 md12>
            <v-progress-linear
                background-color="pink lighten-3"
                color="blue darken-1"
                :indeterminate="true"
            ></v-progress-linear>
            <v-card>
                <h2>留言部分</h2>
            </v-card>
        </v-flex>
        <div>
            <v-add></v-add>
                <div v-for="(msg,index) in messages" :key="index">
                    <v-flex xs12 sm6 md6 offset-sm3>
                        <v-card>
                            <v-message  
                                class="content"
                                :content="msg.content"
                                :author="msg.author"
                                :email="msg.email"
                                :addtime="msg.addtime"
                            />
                            
                            <v-btn 
                                flat 
                                color="pink" 
                                @click="admire(msg)" 
                                round
                                small
                            >
                                <v-icon>favorite</v-icon>{{ msg.admire }}
                            </v-btn>
                        </v-card>
                    </v-flex>
                </div>
        </div>
        <v-btn outline color="blue darken-1" @click="getmsg(startid,count)" :loading="loading" round>查看更多</v-btn>
    </div>
</template>
<script>
import vMessage from './message/eachmessage';
import vAdd from './message/addmessage';
import vNeterror from './global/neterror.vue';
export default {
    components:{
        vMessage,
        vAdd,
        vNeterror
    },
    data:()=>({
        loading: false,
        messages: [
        ],
        startid: 1,//开始获取留言数据id
        count: 4, //每次加载获取的留言条数
    }),
    computed: {
        newmsg: function(){
            let msg = this.$store.state.a.msg;//和有modules时候不一样
            return msg;
        }
    },
    watch: {
        //不能监听this.$store.state.msg: 
        newmsg: function(val){
            this.messages.splice(0,0,val);
            //console.log("监听到了新增留言");//下一步就是把数据发送到服务端存储好了
        }
    },
    methods: {
        getmsg: function(startid,count){
            this.loading = true;
            const _this = this;
            this.$axios({
                method: "GET",
                url: '/msg',
                params: {
                    startid: startid,//前端传递startid，则后端从倒数第startid个数据开始获取
                    count: count
                }
            })
            .then(function(response){
                let msgs = response.data.msgs;
                if(!msgs.length){
                    _this.$message({
                        showClose: true,
                        message: '已全部加载'
                    });
                    _this.loading = false;
                    return;
                }
                for(var i=0;i<msgs.length;i++){
                _this.messages.push(msgs[i]);
                _this.loading = false;
            }
            _this.startid += _this.count;
            })
            .catch(function(error){
                //console.log(error.response.status);
                // console.log('???');
                // console.log(error);
                // console.log('????');
                _this.loading = false;
                _this.$message.error("留言获取错误，请稍后重试");
            })
        },
        admire: function(msg){
            //console.log(this.messages[index].id);
            const _this = this;
            let flag = `msg+${msg.id}`;
            if (localStorage.getItem(flag)){
                this.$message('已经点赞了哦');
            }
            else{
                this.$axios({
                    method: 'put',
                    url: '/msg',
                    params: {
                        admiremsgid: msg.id
                    }//put方法不能params??
                })
                .then(() => {
                    msg.admire ++;
                    localStorage.setItem(flag,'like');
                })
                .catch(err => {
                    _this.$message.error("点赞失败，请稍后重试");
                })
            }
        }
    },
    mounted: async function(){
        this.getmsg(1,this.count);
   
        // try{ 
        //     let response = await fetch('http://47.101.50.208/api/msg?startid=1&count=4'); 
        //     let data = await response.json();
        //     //或者使用axios
        //     // let response = await this.$axios.get('http://47.101.50.208/api/msg?startid=1&count=4');
        //     // let data = response.data;
        //     console.log(data); 
        // } catch(e){ 
        //     console.log("error") 
        // }

    }
}
</script>
<style scoped>
.board{
    text-align: center;
    position: relative;
}
.content{
    margin-top: 10px;
    margin-bottom: 40px; 
}
.first{
    font-weight: bold;
    font-size: 2vw;
}
</style>

