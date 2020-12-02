<template>
  <view class="select-city">
    <scroll-view
      @scroll="scrollTo"
      scroll-y="true" 
      class="select-city-content" 
      :scroll-into-view="scrollIntoViewId" 
      enable-back-to-top="true">
        <view v-for='(item, index) in cityListFormat' :key='index' class="city-item" :id="item.head">
            <view class="city-item-title">{{item.head}}</view>
            <view @tap='selectCity(value)' v-for='(value, key) in item.valueList' :key="key" class="city-item-content">
                <view class="city">{{value.cityName}}</view>
            </view>
        </view>
    </scroll-view>
    <view class="index-content">
        <view v-for="(item, index) in headList" :key="index" @tap="switchIndex(item)" class="index-item" :class="item==scrollIntoViewIdStyle?'active':''">{{item}}</view>
    </view>
  </view>
</template>
<script>
  import Vue from 'vue'
  import {city_list} from "./index"
  let timer = null
  let mode = 'scroll' // scroll、click 滑动左侧城市列表（控制右侧字母的高亮）还是点击右侧字母（控制左侧列表的滚动）
  let totalLength = 0
  let current = 0
  let currentLetter = ''
  export default Vue.extend({
    data() {
      return {
        cityListFormat: [], // 格式化后城市列表
        headList: [], // 首字母缩写列表
        scrollIntoViewId: 'T', // 跳转到的某子元素id
        scrollIntoViewIdStyle: 'T'
      }
    },
    onShow() {
      this.getCityData()
    },
    methods: {
      scrollTo(e) {
        if(mode === 'click') {
          mode = 'scroll'
          return
        }
        if(timer !== null) {
          clearTimeout(timer)
          timer = null
        }
        timer = setTimeout(() => {
          const offsetTop = e.target.scrollTop
          const height = e.target.scrollHeight
          let index = totalLength *  offsetTop/height
          index = Math.ceil(index)
          if(this.headList[index] != this.scrollIntoViewId) {
            current = 0
						this.getLetter(this.cityListFormat, index)
            this.scrollIntoViewIdStyle = currentLetter
          }
        }, 50)
      },
      getLetter(list, index) {
        for(var i = 0; i < list.length; i++) {
            const item = list[i]
            current ++
            if(index === current) {
              currentLetter = item.head
              break
            }
            for(var j = 0; j < item.valueList.length; j++) {
              current ++
              if(index === current) {
                currentLetter = item.head
                return
              }
            }
          }
      },
      // 获取城市数据
      getCityData() {
        const cityList = city_list
        totalLength = cityList.length
        const cityListNew = []
        for(let elem of cityList.values()) {
            const head = elem.headcharacter
            const index = this.headList.indexOf(head)
            if (index <= -1) {
                this.headList.push(head)
                cityListNew.push({
                    head,
                    valueList: [ elem ]
                })
            } else {
                cityListNew[index].valueList.push(elem)
            }
        }
        // 排序
        this.headList = this.headList.sort()
        this.cityListFormat = cityListNew.sort((a, b) => {
            return a.head.charCodeAt() - b.head.charCodeAt()
        })
        this.scrollIntoViewId = this.headList[0]
        this.scrollIntoViewIdStyle = this.headList[0]
        totalLength = totalLength + this.headList.length
      },
      // 切换城市首字母
      switchIndex(data) {
        mode = 'click'
        this.scrollIntoViewId = this.scrollIntoViewIdStyle
        setTimeout(() => {
          this.scrollIntoViewId = data
          this.scrollIntoViewIdStyle = data
        }, 0)
      },
      // 选择城市
      selectCity(data) {
          const cityCode = data.cityCode
          const adCode = data.areacode
          const cityName = data.cityName
          uni.$emit('selectedCity', {cityCode, cityName})
          uni.navigateBack({
             delta: 1
          });
      }
    },
  })
</script>
<style lang="less" scoped>
view, text, scroll-view {
        box-sizing: border-box;
}
.select-city {
    width: 100vw;
    height: 100vh;
    padding-top: 70upx;
    .city-top {
      background: #fafafa;
      padding: 0 0 0 40rpx;
      position: fixed;
      height: 70upx;
      line-height: 70upx;
      top: 0upx;
      vertical-align: middle;
      z-index: 5;
      width: 100%;
      font-weight: bold;
      font-family: PingFangSC-Semibold;
      font-size: 30rpx;
      color: #141E25;
      letter-spacing: 0;
      border-bottom: 2rpx solid #EFEFEF;
    }
    .select-city-content {
        height: 100%;
        .city-item {
            .city-item-title {
                background: #F5F5F5;
                height: 50rpx;
                line-height: 50rpx;
                font-family: PingFangSC-Semibold;
                font-size: 24rpx;
                color: #778690;
                padding-left: 40rpx;
            }
            .city-item-content {
                padding-left: 40rpx;
                .city {
                    font-family: PingFangSC-Semibold;
                    font-size: 24rpx;
                    color: #141E25;
                    letter-spacing: 0;
                    height: 80rpx;
                    line-height: 80rpx;
                }

            }
        }
    }
    .index-content {
        position: fixed;
        right: 16rpx;
        top: 50%;
        transform: translateY(-50%);
        .index-item {
            width: 26rpx;
            height: 26rpx;
            border-radius: 50%;
            font-family: PingFangSC-Semibold;
            font-size: 20rpx;
            letter-spacing: 0;
            text-align: center;
            line-height: 26rpx;
            margin-top: 4rpx;
            color: #778690;
            &.active {
                color: #FFFFFF;
                background: #6034FF;
            }
        }
    }
}
</style>