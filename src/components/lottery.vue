<template>
    <div class="hello">
        <div class="lottery-box" id="app">
            <h1 class="title">Tina九宫格抽奖</h1>
            <div class="lottery">
                <div class="lottery-item">
                    <div class="lottery-start">
                        <div class="box gray" v-if="isStart===0">
                            <p>活动未开始</p>
                        </div>
                        <div class="box" @click="startLottery" v-if="isStart===1">
                            <p><b>抽奖</b></p>
                            <!--                            <p>消耗{{ score }}积分</p>-->
                            <p>点击我</p>
                        </div>
                        <div class="box gray" v-if="isStart===2">
                            <p>活动已过期</p>
                        </div>
                    </div>
                    <ul>
                        <li v-for="(item,i) in awards" :key="i" :class="i==index?'on':''">
                            <div class="box">
                                <p><img :src="item.img" :alt="item.name"/></p>
                                <p>{{ item.name }}</p>
                            </div>
                        </li>
                    </ul>
                </div>
            </div>
            <!-- 中奖弹窗 -->
            <div class="mask" v-if="showToast"></div>
            <div class="lottery-alert" v-if="showToast">
                <h1>恭喜您</h1>
                <p><img :src="awards[index].img" alt=""></p>
                <h2>获得{{ awards[index].name }}</h2>
                <div class="btnsave" @click="showToast=false">确定</div>
            </div>
        </div>

    </div>
</template>

<script>

import axios from "axios";

export default {
    name: 'lottery',
    props: {},
    data() {
        return {
            isStart: 1,
            score: 10, //消耗积分
            awards: [],   //奖品1-9
            index: -1,  // 当前转动到哪个位置，起点位置
            count: 8,  // 总共有多少个位置
            timer: 0,  // 每次转动定时器
            speed: 200,  // 初始转动速度
            times: 0,    // 转动次数
            cycle: 50,   // 转动基本次数：即至少需要转动多少次再进入抽奖环节
            prize: -1,   // 中奖位置
            click: true,
            showToast: false, //显示中奖弹窗
            award: {}
        }
    },
    created() {
        this.e()
    },
    methods: {
        startLottery() {
            this.f()
            if (!this.click) {
                return
            }
            this.startRoll();
        },
        // 开始转动
        startRoll() {
            this.times += 1 // 转动次数
            this.oneRoll() // 转动过程调用的每一次转动方法，这里是第一次调用初始化
            // 如果当前转动次数达到要求 && 目前转到的位置是中奖位置
            if (this.times > this.cycle + 10 && this.prize === this.index) {
                clearTimeout(this.timer)  // 清除转动定时器，停止转动
                this.prize = -1
                this.times = 0
                this.speed = 200
                this.click = true;
                var that = this;
                setTimeout(res => {
                    console.log(res)
                    that.showToast = true;
                }, 500)
            } else {
                if (this.times < this.cycle) {
                    this.speed -= 10  // 加快转动速度
                } else if (this.times === this.cycle) {
                    //中奖位置,可由后台返回
                    this.prize = this.award.orderNum;
                    if (this.prize > 7) {
                        this.prize = 7
                    }
                } else if (this.times > this.cycle + 10 && ((this.prize === 0 && this.index === 7) || this.prize === this.index + 1)) {
                    this.speed += 110
                } else {
                    this.speed += 20
                }
                if (this.speed < 40) {
                    this.speed = 40
                }
                this.timer = setTimeout(this.startRoll, this.speed)
            }
        },

        // 每一次转动
        oneRoll() {
            let index = this.index // 当前转动到哪个位置
            const count = this.count // 总共有多少个位置
            index += 1
            if (index > count - 1) {
                index = 0
            }
            this.index = index
        },
        e() {
            axios.get("http://tinaroot.cn/award/e")
                .then(response => {
                    this.awards = response.data
                }, err => {
                    console.log(err)
                })
        },
        f() {
            axios.get("http://tinaroot.cn/award/f")
                .then(response => {
                    this.award = response.data
                }, err => {
                    console.log(err)
                })
        }
    }
}
</script>


<style scoped>

</style>
