<template>
    <div  style="margin: 100px 18% 40px 18%">
            <honor-title :get-all-data="handleGetHonour" @emitSearch="onSearchSingle" :title="title" english="Company Honors"  :ismany="false"  ></honor-title>
            <honor-item v-if="isLoading&&!isSearchData" :honour-data-list="honourDataList" style="margin-bottom: 30px;"></honor-item>
                
                <!-- <search-item v-if="isSearchData" v-for="(item.index) in honourDataList" :key="index" :search-info="item"></search-item> -->
                <search-item v-if="isSearchData" v-for="(item,index) in honourDataList"  :search-info="item" :key="index" ></search-item>
            

            <div v-show="!isLoading" style="min-height: 500px;position: relative;" >
                <loading></loading>
            </div>
            <div v-show="!isNoData&&isLoading" style="min-height: 400px;position: relative;">
                <no-data></no-data>
            </div>
            <el-pagination
                   background 
                  :page-size="pageSize"
                  :total="total"
                  :current-page="current"
                  layout="total,sizes,prev,pager,next,jumper"
                  :page-sizes="pageSizes"
                  prev-text="上一页"
                  next-text="下一页"
                  @size-change="handleSizeChange"
                  @current-change="handleCurrentChange">
                </el-pagination>
    </div>
</template>
<script>
import {HonorItem } from './index'
import HonorTitle from '@/components/title'
import { getHonours } from "@/api/honor"
import Long from "long"
import Loading from "@/components/loading"
import NoData from "@/components/noData"
import {searchArticles} from "@/api/home"
import SearchItem from "@/components/search"
export default {
    data(){
        return {
            pageSize:10,
            current:1,
            total:0,
            pageSizes:[8,16,32],
            title:{
                nameLeft:'荣誉',
                nameRight:'中亚'
            },
            isLoading:false,
            isNoData:false,
            honourDataList:[],//列表数据
            fastsearch:null,
            isSearchData:false,
            
        }
    },
    methods:{
        // pagesize 变化回调
        handleSizeChange(val){
            this.pageSize = val
            this.handleGetHonour()
         },
       //current 变化回调
        handleCurrentChange(val){
            this.current = val
            this.handleGetHonour()
        },
        handleGetHonour({current=this.current,pageSize=this.pageSize}={}){
            getHonours({current,pageSize}).then(response => {
                this.isLoading = true
                this.isSearchData = false
                if(response.value && response.value.length >0){
                    this.isNoData = true
                    this.total = response.page.total
                    this.honourDataList = response.value.map(item=>{
                        item.id = (Long.fromValue(item.id)).toString()
                        return item
                    })
                }
            })
        },
        onSearchSingle(val){
            this.fastsearch = val
            this.handleSearch()
        },

        
        //搜索数据 
        handleSearch({current=this.current,pageSize=this.pageSize,articletype=5,fastsearch=this.fastsearch}={}){
            this.isLoading = false
            this.isNoData = false
            this.isSearchData = true
            this.honourDataList = []
        const data = {
            current,
            pageSize,
            articletype,
            fastsearch
        }
        searchArticles(data).then(response => {
            this.isLoading = true
            this.total = response.articles.page.total
            this.honourDataList = response.articles.value
            if(response.articles.value && response.articles.value.length>0){
                this.isNoData = true
            }
            
        })
       },
    },
    components:{
        HonorTitle,
        HonorItem,
        NoData,
        Loading,
        SearchItem
    },
   
    mounted(){
        this.handleGetHonour()
    }
}
</script>