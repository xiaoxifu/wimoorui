<template>
	<div class="main-sty gary-bg">
		<el-row>
			 <el-col :xl="4" :lg="2"  class="ri-tabs">
				<el-tabs tab-position="left"  v-model="activeName">
				    <el-tab-pane label="基本信息" name="1">
						<template #label>
							 <el-link :underline="false"  href="#base">基本信息</el-link>
						</template>
					</el-tab-pane>
					<el-tab-pane  label="组合信息" name="2">
						<template #label>
							 <el-link v-if="issfg=='1'" :underline="false"  href="#children">组合信息</el-link>
							 <el-link v-else-if="issfg=='2'" :underline="false"  href="#parent">组合信息</el-link>
							 <el-link v-else :underline="false"  href="#normal">组合信息</el-link>
						</template>
					</el-tab-pane>
				    <el-tab-pane label="采购信息" name="3">
						<template #label>
							 <el-link :underline="false" href="#purchase">采购信息</el-link>
						</template>
					</el-tab-pane>
					<el-tab-pane label="规格信息" name="4">
						<template #label>
							 <el-link :underline="false" href="#specs">规格信息</el-link>
						</template>
					</el-tab-pane>
					<el-tab-pane label="辅料关联" name="5">
						<template #label>
							 <el-link :underline="false" href="#consumables">辅料关联</el-link>
						</template>
					</el-tab-pane>
				    <el-tab-pane label="物流信息" name="6">
						<template #label>
							 <el-link :underline="false" href="#logistics">物流信息</el-link>
						</template>
					</el-tab-pane>
					<el-tab-pane label="库存信息" name="7">
						<template #label>
							 <el-link :underline="false" href="#inv">库存信息</el-link>
						</template>
					</el-tab-pane>
				  </el-tabs>
			</el-col>
			<el-col  :xl="16" :lg="21">
				<el-card class="fr-con">
					 <el-scrollbar class="he-scr-car" @scroll="scroll">
						 <Listinfo ref="infoRef" />
					</el-scrollbar>
						<div class='text-center mar-top-16'>
							<el-space>
								<el-button @click="closeTab" >关闭</el-button>
								<el-button @click.stop="copyinfo" type="primary">复制</el-button>
								<el-button @click.stop="editinfo" type="primary">编辑</el-button>
							</el-space>
						</div>
						
				</el-card>
			</el-col>
		</el-row>
	</div>
</template>

<script setup>
import { ref,reactive,onMounted,watch,onUnmounted,inject} from 'vue'
import tabScroll from"@/utils/tab_scroll"
import Listinfo from"./components/listinfo"
import {useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import materialApi from '@/api/erp/material/materialApi.js';
	    const emitter = inject("emitter"); // Inject `emitter`
		let activeName =ref('1')
		const infoRef=ref({});
		let router = useRouter()
		let issfg=ref("0");
		const mid=router.currentRoute.value.query.details;
		function scroll(obj){
			activeName.value = tabScroll(obj,"tab-scroll")
		}
		 function closeTab(){
		 	emitter.emit("removeTab", 100);
		 }
		 function loadData(){
		 	materialApi.getMaterialInfo({"id":mid}).then((res)=>{
				if(res.data){
					infoRef.value.loadData(res.data);
					issfg.value=res.data.material.issfg;
				}
			})
		}
		function editinfo(){
			router.push({
				path:'/localproduct/editinfo',
				query:{
				  title:"编辑产品",
				  path:'/localproduct/editinfo',
				  details:mid,
				}
			}) 
		}
		function copyinfo(){
			router.push({
				path:'/localproduct/editinfo',
				query:{
				  title:"编辑产品",
				  path:'/localproduct/editinfo',
				  details:mid,
				  iscopy:'ok'
				}
			}) 
		}
		onMounted(()=>{
			loadData();
		})
</script>

<style>
	
.small-tab  th.el-table__cell{
	background-color: #fff;
}	
	.wi-60{
		width:60px;
	}
.mar-top-16{
	margin-top: 16px;
}
.fr-con .el-divider{
	padding-bottom:8px!important;
}
   .fr-con h3{
   	margin-bottom:16px ;
   }
	.fr-con .el-card__body{
		height:calc(100vh - 90px);
		min-height:600px;
	}
	.ri-tabs{
		display: flex;
		justify-content: flex-end;
	}
	.ri-tabs .el-tabs--left .el-tabs__nav-wrap.is-left::after{
		height:0px;
	}
	.in-wi-24{
		width: 400px;
	}
	.flex-center .el-link--inner{
		display: flex;
		align-items: center;
		margin-left: 8px;
		opacity: 0;
	}
	.grid-row:hover .flex-center .el-link--inner{
		opacity: 1;
	}
	.fo-we-400{
		font-weight: 400;
	}
	.he-scr-car{
		height:calc(100vh - 176px)
	}
</style>
