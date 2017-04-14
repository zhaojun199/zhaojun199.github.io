<template>
    <div class="wrapper" @click.stop>
        <div class="singer-list">
            <ul class="singer-ul">
                <template v-for="item in cur_list">
                    <li class="singer-li" @click="chooseSinger(item.id, item.singer_name)" style="width:100%">
                        <div class="singer-it-top" style="width:100%">
                            <img v-lazy="item.icon">
                        </div>
                        <div class="singer-it-bottom">{{item.title}}</div>
                    </li>
                </template>
            </ul>
        </div>
    </div>
</template>
<script>
import Vue from 'vue'
import lazy from '@/javascript/lazy'
Vue.use(lazy)

export default {
    data() {
            return {
                singer_url: './static/lazy.json',
                index: '',
                cur_list: [],
            }
        },
        created() {
          this.getSingerlist();
        },
        methods: {
            getSingerlist() {
                this.axios.get(this.singer_url).then(res => {
                    if (res.data.data) {
                        this.cur_list = this.cur_list.concat(res.data.data);
                    } else {
                        this.$message({
                            type: 'error',
                            message: res.data.message
                        });
                    }
                })
            },
            changeIndex(e) {
                let cur = e.srcElement || e.target;
                if (this.index === cur.innerText) return;
                this.index = cur.innerText;
                this.list.forEach((val) => {
                    if (val[0]['c_index'] === cur.innerText) {
                        this.cur_list = val;
                        return false;
                    }
                })
            },
            chooseSinger(id, name) {
                this.index = 123;
                this.$emit('singer', id, name);
            },
            resetIndex() {
                this.$emit('singer', null);
            }
        }
}
</script>
