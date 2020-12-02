<template>
    <div 
        class="area-picker" 
        :style="{width: typeof width === 'number' ? width + 'px' : width}"
    >
        <section class="area-picker-header">
            <div   
                class="area-picker-header-index"
                v-for="item in renderIndexs"
                :key="item.index"
                :class="{active: curIndex === item.index}"
                @click.stop="onClickIndex(item.index)"
            >{{item.label}}</div>
        </section>
        <section class="area-picker-wrap" :style="{height: height + 'px'}">
            <transition :name="transitionName" >
                <section class="area-picker-wrap-item" v-if="curIndex === 1" key="province"> 
                        <ul class="area-picker-container">
                            <li 
                                class="area-picker-container-item"
                                v-for="(item) in provinceList"
                                :key="item.code"
                                :class="{active: item.code == province}"
                                @click="onPickProvince(item)"
                            >{{item.name}}</li>
                        </ul>
                </section>
                <section class="area-picker-wrap-item" v-if="curIndex === 2" key="city"> 
                        <ul class="area-picker-container">
                            <li 
                                class="area-picker-container-item"
                                v-for="(item) in cityList"
                                :key="item.code"
                                :class="{active: item.code == city}"
                                @click="onPickCity(item)"
                            >{{item.name}}</li>
                            <div class="area-picker-container-empty" v-show="cityList.length === 0">暂无数据</div>
                        </ul>
                </section>
                <section class="area-picker-wrap-item" v-if="curIndex === 3" key="county"> 
                        <ul class="area-picker-container">
                            <li 
                                class="area-picker-container-item"
                                v-for="(item) in countyList"
                                :key="item.code"
                                :class="{active: item.code == county}"
                                @click="onPickCounty(item)"
                            >{{item.name}}</li>
                            <div class="area-picker-container-empty" v-show="countyList.length === 0">暂无数据</div>
                        </ul>
                </section>
            </transition>
        </section>
    </div>
</template>

<script>
import { getAreaList } from './areaData.js'
const area = getAreaList();
export default {
    name: 'areaPicker',

    model: {
        prop: 'value',
        event: 'change'
    },

    props: {
        width: {
            type: [Number, String],
            default: 360,
        },

        height: {
            type: Number,
            default: 290
        },

        // 地区的列数范围，默认3列，省/市/区
        columns: {
            type: Number,
            default: 3,
            validator: function (value) {
                return +value > 0 && +value <= 3;
            }
        },

        value: {
            type: Array,
        }
    },

    data(){
        return {
            indexs: [
                { index: 1, label: '省份' },
                { index: 2, label: '城市' },
                { index: 3, label: '县区' }
            ],
            transitionName: 'to-left',

            curIndex: 1,
            provinceList: this.getList('province'),
            province: '',
            city: '',
            county: '',

        }
    },

    computed: {
        renderIndexs(){
            return this.indexs.filter((item) => item.index <= (+this.columns));
        },
        
        cityList(){
            if(this.province){
                return this.getList('city', this.province);
            }else{
                return []
            }
        },

        countyList(){
            if(this.city){
                return this.getList('county', this.city);
            }else{
                return []
            }
        }
    },

    watch: {
        value: {
            handler(v){
                if(!Array.isArray(v)){
                    return
                }
                if(v[0]){
                    this.province = v[0];
                }
                if(v[1]){
                    this.city = v[1];
                }
                if(v[2]){
                    this.county = v[2]
                }
            },
            immediate: true
        },
    },

    methods: {
        // 根据参数返回对应的地址数据， type: province, city, county
        getList(type, code = ''){
            let res = [];
            let areaData = {};
            let keyCode = '';

            if(!code){
                areaData = area.province_list;
                res = Object.keys(areaData).map((code) => {
                    return {
                        code,
                        name: areaData[code]
                    }
                })

                return res
            }

            switch (type) {
                case 'city':
                    areaData = area.city_list;
                    keyCode = String(code).slice(0, 2);
                    break;
                case 'county':
                    areaData = area.county_list;
                    keyCode = String(code).slice(0, 4);
                    break;
            }

            res = Object.keys(areaData).filter((code) => {
                return String(code).startsWith(keyCode)
            }).map((code) => {
                return {
                    code,
                    name: areaData[code]
                }
            })

            return res
        },

        onPickProvince(province){
            if(this.province != province.code ){
                this.province = province.code;
                this.city = '';
                this.county = '';
            }

            const res = this.formatArea();
            this.$emit('pick', res.areaObjs);
            this.$emit('pick-province', province);
            this.$emit('change', res.codes);
            if(this.curIndex < this.columns){
                this.transitionName = 'to-left';
                this.curIndex++
            }
        },

        onPickCity(city){
            if(this.city != city.code ){
                this.city = city.code;
                this.county = '';
            }

            const res = this.formatArea();
            this.$emit('pick', res.areaObjs);
            this.$emit('pick-city', city);
            this.$emit('change', res.codes);
            if(this.curIndex < this.columns){
                this.transitionName = 'to-left';
                this.curIndex++
            }
        },

        onPickCounty(county){
            if(this.county != county.code ){
                this.county = county.code;
            }

            const res = this.formatArea();
            this.$emit('pick', res.areaObjs);
            this.$emit('pick-county', county);
            this.$emit('change', res.codes);
            if(this.curIndex < this.columns){
                this.transitionName = 'to-left';
                this.curIndex++
            }
        },

        formatArea(){
            let indexs = ['province', 'city', 'county'];
            let codes = new Array(this.columns);
            let areaObjs = new Array(this.columns);

            for (let i = 0; i < this.columns; i++) {
                codes[i] = this[indexs[i]] || undefined;
                areaObjs[i] = { code: codes[i], name: area[`${indexs[i]}_list`][+codes[i]] };
            }
            return {
                codes,
                areaObjs
            }
        },

        // 获取当前地址的中文名，返回一个数组
        getAreaNameList(codeList){
            let res = [];
            if(!Array.isArray(codeList)){
                return res
            }
            if(codeList[0]){
                res[0] = area.province_list[+codeList[0]] || '';
            }
            if(codeList[1]){
                res[1] = area.city_list[+codeList[1]] || '';
            }
            if(codeList[2]){
                res[2] = area.county_list[+codeList[2]] || '';
            }
            return res
        },
        
        onClickIndex(index){
            if(this.curIndex === index){
                return
            }
            if(this.curIndex < index){
                this.transitionName = 'to-left';
            }else{
                this.transitionName = 'to-right';
            }
            this.curIndex = index;
        }
    }
}
</script>

<style lang='scss' scoped>
// 使用的scss
$theme-color: #FF7B03;
$fc1: #383838;
$fc2: #515151;
$fc3: #999999;

ul,li{
    margin: 0;
    padding: 0;
}
li{
    list-style: none;
}

.area-picker{
    background-color: #ffffff;
    border-radius: 4px;
    box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);

    &-header{
        display: flex;
        justify-content: flex-start;
        align-items: flex-start;
        border-radius: 4px 4px 0 0;
        &-index{
            flex-grow: 1;
            padding: 14px 0px;
            text-align: center;
            font-size: 16px;
            font-weight: 400;
            color: $fc3;
            background-color: #F8F8F8;
            cursor: pointer;
            &.active{
                color: $theme-color;
                background-color: #ffffff;
            }
            &:first-child{
                border-radius: 4px 0 0 0;
            }
            &:last-child{
                border-radius: 0 4px 0 0;
            }
        }
    }

    &-wrap{
        overflow-y: auto;
        overflow-x: hidden;
        position: relative;

        &::-webkit-scrollbar {
            background-color: transparent;
            width: 6px;
            background-clip: padding-box;
        }
        &::-webkit-scrollbar-thumb {
            background-color: #dddee0;
            border-radius: 6px;
        }
        
        &-item{
            width: 100%;
            position: absolute;
        }

        .area-picker-container{
            padding: 28px 24px;
            display: flex;
            justify-content: flex-start;
            align-items: flex-start;
            align-content: flex-start;
            flex-wrap: wrap;
            &-item{
                font-size: 14px;
                font-weight: 400;
                color: $fc1;
                padding: 4px 6px;
                margin: 8px;
                cursor: pointer;
                &.active{
                    background-color: $theme-color;
                    color: #ffffff;
                }
            }
            &-empty{
                text-align: center;
                color: $fc3;
                font-size: 14px;
                padding-top: 12px;
                width: 100%;
            }
        }
    }
}

// 切换动画
.to-left-enter-active, 
.to-left-leave-active,
.to-right-enter-active, 
.to-right-leave-active {
  transition: all .2s ease;
}
.to-left-enter{
  transform: translateX(50%);
  opacity: 0;
}
.to-left-leave-to{
    transform: translateX(-50%);
    opacity: 0;
}
.to-right-enter{
  transform: translateX(-50%);
  opacity: 0;
}
.to-right-leave-to{
    transform: translateX(50%);
    opacity: 0;
}
.to-left-enter-to,
.to-left-leave,
.to-right-enter-to,
.to-right-leave{
    transform: translateX(0);
    opacity: 1;
}

</style>