<template>
	<div class="main-sty">
		<div class="con-header">
					
			<el-row>
				<Group @change="getGroup" isproduct="ok" />
				<el-space>
					<Status @status="getStatus" />
					<Owner @owner="getOwner" />
					<el-select v-model="disabletype" @change='disabletypeChange' placeholder="全部显示状态"
						style="width: 110px">
						<el-option label="未隐藏" value="false">未隐藏</el-option>
						<el-option label="隐藏" value="true">隐藏</el-option>
						<el-option label="失效" value="invalid">失效</el-option>
						<el-option label="不限" value="all">不限</el-option>
					</el-select>
					<el-input v-model="searchKeywords"  @input="changeKeywords"  placeholder="请输入" class="input-with-select">
						<template #prepend>
							<el-select v-model="searchtype" @change='searchTypeChange' placeholder="SKU"
								style="width: 110px">
								<el-option label="SKU" value="sku">SKU</el-option>
								<el-option label="同级SKU" value="childsku">同级SKU</el-option>
								<el-option label="本地SKU" value="msku">本地SKU</el-option>
								<el-option label="ASIN" value="asin">ASIN</el-option>
								<el-option label="同级ASIN" value="childasin">同级ASIN</el-option>
								<el-option label="父ASIN" value="parentasin">父ASIN</el-option>
							</el-select>
						</template>
						<template #append>
							<el-button @click="searchConfirm">
								<el-icon style="font-size: 16px;align-itmes:center">
									<search />
								</el-icon>
				  	</el-button>
						</template>
					</el-input>
					<el-popover   :popper-append-to-body="false" v-model:visible="moreSearchVisible" :width="400" trigger="click">
						<template #reference>
							<el-button title='高级筛选' class='ic-btn'>
								<menu-unfold theme="outline" size="16" :strokeWidth="3" />
							</el-button>
						</template>
						 <el-form :model="form"  label-width="auto"  label-position="left">
							<el-form-item label="产品标签">
							     <Tags @change="changeTag"  />
							   </el-form-item>
						    <el-form-item label="产品分类">
								 <Category @change="getCategory" />
							 </el-form-item>
							<el-form-item label="销量">
							     <el-select  v-model="salerangecheck"  :teleported="false" placeholder="请选择" >
							    <el-option
							      v-for="item in salerange"
							      :key="item.value"
							      :label="item.label"
							      :value="item.value"
							    />
							     </el-select>
							   </el-form-item>
							   <el-form-item label="配送方式">
							        <el-select  v-model="isfba"  :teleported="false" placeholder="请选择" @change="trantypeChange">
							       <el-option
							         v-for="item in saleslist"
							         :key="item.value"
							         :label="item.label"
							         :value="item.value"
							       />
							        </el-select>
							      </el-form-item>
								  <el-form-item label="产品名称">
								    <el-input v-model="proname" clearable="true"></el-input>
								    </el-form-item>
								  <el-form-item label="备注">
									  <el-input v-model="remark" clearable="true"></el-input>
								    </el-form-item>
									<el-form-item label="发现差评">
										<el-checkbox v-model="isbadreview" label="" size="large" />
									 </el-form-item>
									
								    <el-form-item>
								  <el-button type="primary" @click="submitSearch(formRef)">搜索</el-button>
								  <el-button @click="resetSearch(formRef)">取消</el-button>
								    </el-form-item>
								
						</el-form>
						
					  </el-popover>
					
					<el-popover    v-model:visible="dataSearchVisible" :width="400" trigger="click">
						<template #reference>
							<el-button>筛选器</el-button>
						</template>
						 <el-form :model="form" label-width="auto"   >
							<el-form-item label="数据字段">
							     <el-select  v-model="dataColumn"  :teleported="false" placeholder="请选择" @change="columnChange">
									<el-option
									  v-for="item in columnList"
									  :key="item.value"
									  :label="item.label"
									  :value="item.value"
									/>
							     </el-select>
							   </el-form-item>
							   <el-form-item label="符号">
							       <el-radio-group v-model="mark">
							           <el-radio :label="3">大于</el-radio>
							           <el-radio :label="6">小于</el-radio>
							           <el-radio :label="9">等于</el-radio>
							         </el-radio-group>
							      </el-form-item>
								  <el-form-item label="数值">
									  <el-input v-model="dataValue" type="number"  placeholder="用%时请输入小数"></el-input>
									  <el-button style="margin-top: 3px;" @click="addColumn" type="success">添加条件</el-button>
									   <el-button style="margin-top: 3px;" @click="clearColumn" type="success">清空条件</el-button>
								    </el-form-item>
									<el-form-item label="筛选条件">
										{{columntext}}
									</el-form-item>
								    <el-form-item>
								  <el-button type="primary" @click="submitDataForm(formRef)">搜索</el-button>
								  <el-button @click="resetDataForm(formRef)">取消</el-button>
								    </el-form-item>
						</el-form>
					  </el-popover>
					<el-button>重置</el-button>
				</el-space>
			</el-row>
			<!--功能区域 -->
			<el-row>
				<el-space>
					<el-button type="primary" class="im-but-one" @click.stop="showRefreshDialog">
					    <plus theme="outline" size="18" fill="#fff" :strokeWidth="4"/>
					    <span>同步商品</span>
					</el-button>
					<el-button @click.stop="refreshFBAInv">批量同步FBA库存</el-button>
					<!-- <el-button @click="modifyPrice">批量调价</el-button> -->
					<!-- <el-dropdown trigger="click">
					  <el-button>
					             配对操作<el-icon class="el-icon--right"><arrow-down /></el-icon>
					  </el-button>
					  <template #dropdown>
					    <el-dropdown-menu >
					      <el-dropdown-item>配对</el-dropdown-item>
					      <el-dropdown-item>解除配对</el-dropdown-item>
					    </el-dropdown-menu>
					  </template>
					</el-dropdown> -->
					<el-dropdown trigger="click">
					  <el-button>
					             导入<el-icon class="el-icon--right"><arrow-down /></el-icon>
					  </el-button>
					  <template #dropdown>
					    <el-dropdown-menu >
						  <el-dropdown-item>其他成本</el-dropdown-item>
					    </el-dropdown-menu>
					  </template>
					</el-dropdown>
				</el-space>
				<div class='rt-btn-group'>
					<el-space>
					<el-dropdown :hide-on-click="false"   @command="handleCommand">
					    <el-button class='ic-btn'  title='排序'>
					       <sort-one theme="outline" size="16"  :strokeWidth="3"/>
					    </el-button>
					    <template #dropdown>
							  <el-scrollbar height="calc(100vh - 220px)">
					      <el-dropdown-menu >
							<el-dropdown-item disabled>排序依据</el-dropdown-item>
					        <el-dropdown-item v-for="(item,index) in rankData"  @click="rankChange(item.value)"
							 :class="{r_active:currentRank==item.value}" 
							>{{item.name}}</el-dropdown-item>
					        <el-dropdown-item divided  disabled>排序循序</el-dropdown-item>
					        <el-dropdown-item v-for="item in soltData" @click="soltChange(item.value)" 
							:class="{r_active:currentSolt==item.value}">{{item.name}}</el-dropdown-item>
					      </el-dropdown-menu>
						    </el-scrollbar>
					    </template>
					  </el-dropdown>
					  <el-button class='ic-btn' title="日均销量公式" @click="EditSaleFormula">
						  <el-icon class="font-medium"><Operation /></el-icon>
					  </el-button>
					</el-space>
					<el-button class='ic-btn' title='帮助文档'>
						<help theme="outline" size="16" :strokeWidth="3" />
					</el-button>
					 
				</div>
			</el-row>
			
			<!-- <el-row>
			筛选器
				<el-tag  closable>库存 > 10</el-tag>
			</el-row> -->
		</div>
		
		<div class="con-body">
			
			<Table @checkRow="checkRows"  ref="tableRef"  />
		</div>
	</div>
	<BatchModifypriceDialog ref="priceDialogRef"/>
	<!-- 日均销量公式 -->
	<SaleFormula ref = "SaleFormulaRef"/>
	<RefreshProDialog ref="refreshRef" />
</template>

<script>
	import {ref,reactive,onMounted,watch,h} from 'vue';
	import {Help,Plus,MenuUnfold,SettingTwo,SortOne} from '@icon-park/vue-next';
	import {ElMessage,ElDivider} from 'element-plus';
	import {Search,ArrowDown,Check,Operation} from '@element-plus/icons-vue';
	import Table from"./components/table"
	import BatchModifypriceDialog from"./components/batchmodifypriceDialog.vue"
	import SaleFormula from"./components/sale_formula"
	import Group from './components/group.vue';
	import Owner from '@/components/header/owner.vue';
	import Tags from '@/components/header/tags.vue';
	import Category from '@/components/header/category.vue';
	import Status from './components/prostatus.vue';
	import RefreshProDialog from './components/refreshProduct.vue';
	import productRefreshApi from '@/api/amazon/product/productRefreshApi.js';

	export default {
		name:'Index',
		components: {
			Help,ArrowDown,SortOne,
			Search,MenuUnfold,Check,
			Plus, SettingTwo,Table,
			BatchModifypriceDialog,Group,Owner,Status,Tags,Category,
			Operation,
			SaleFormula,
			RefreshProDialog,
		},
		emits:["getdata"],
		setup(prop,context){
			onMounted(()=>{

			})
			let refreshRef=ref();
			let priceDialogRef =ref()
			let radio2 = ref('ASIN')
			let salerangecheck =ref("")
			let trantype=ref()
			let currentRank =ref('sku')
			let currentSolt =ref("asc")
			let isload=ref(true)
			let searchtype=ref("sku")
			let disabletype=ref("false")
			let isfba=ref("")
			let tableRef=ref();
			let searchKeywords=ref();
			let remark=ref();
			let proname=ref();
			let SaleFormulaRef =ref()
			let moreSearchVisible=ref(false);
			let dataSearchVisible=ref(false);
			let isbadreview=ref(false); 
			let mark=ref(3);
			let dataValue=ref();
			let columntext=ref("");
			let columnval=ref("");
			let tags=ref();
			let saleslist=reactive([{"value":"","label":"不限"},{"value":"AMAZON","label":"亚马逊配送"},{"value":"DEFAULT","label":"卖家自配送"}])
			let columnList=reactive([{"value":"v.advacos","label":"ACOS"},{"value":"v.advctr","label":"CTR"},
			{"value":"v.acoas","label":"ACOAS"},{"value":"v.advspc","label":"CR"},
			{"value":"v.sessionrate","label":"七日转化率"},{"value":"v.proprate","label":"净利润率"},
			{"value":"v.dayfulfilla","label":"可售天数"},{"value":"v.afn_fulfillable_quantity","label":"库存"},{"value":"v.profitrate","label":"产品利润率"}]);
			let dataColumn=ref("v.advacos");
	    let props = { multiple: true }
		let soltData=reactive([{name:'从高到低',value:'desc'},{name:'从低到高',value:"asc"}])
		var queryParam={};
		let rankData =reactive([
			{
				name:'SKU',
				value:'sku',
			},
		{
			name:'日均销量',
			value:'averageSalesDay',
		},
		
		{
			name:'库存',
			value:'afn_fulfillable_quantity',
		},
		{
			name:'可售天数',
			value:'dayfulfilla',
		},
		{
			name:'利润',
			value:'profits',
		},
		{
			name:'价格',
			value:'landed_amount',
		},
		{
			name:'访问量',
			value:'sessions',
		},
		{
			name:'点击量',
			value:'advclick',
		},
		{
			name:'广告花费',
			value:'advspend',
		},
		{
			name:'净利润',
			value:'profitall',
		},
		{
			name:'产品利润率',
			value:'newprorate',
		},
		{
			name:'上架日期',
			value:'opendate',
		},
		{
			name:'Review数量',
			value:'feedback_count',
		},
		{
			name:'Review评分',
			value:'positive_feedback_rating',
		},
		{
			name:'父ASIN',
			value:'parentAsin',
		},
		])
		let salerange =[
			{label:'不限',value:""},{label:'销量小幅下降',value:"-3"},
			{label:'销量大幅下降',value:"-25"},{label:'销量小幅上升',value:"3"},
			{label:'销量大幅下降',value:"25"}
		]
		let options =ref([]);
		let selection = ref([])
		function checkRows(rows){
			selection.value =  rows;
		}
		function modifyPrice(){
			if(selection.value.length>0){
				priceDialogRef.value.priceVisable =true
			}else{
				ElMessage({
					message:'请选择商品',
					type:'error',
				})
			}
			
		}
		//排序
		function rankChange(val){
			currentRank.value= val;
			queryParam.sort=val;
			queryParam.order=currentSolt.value;
			refreshTable();
		}
 
		function soltChange(val){
			currentSolt.value =val;
			queryParam.sort=currentRank.value;
			queryParam.order=val;
			refreshTable();
		}
		function getGroup(obj){
			queryParam.groupid=obj.groupid;
			queryParam.marketplaceid=obj.marketplaceid;
			queryParam.searchtype=searchtype.value;
			queryParam.isfba=isfba.value;
			if(remark.value && remark.value!=""){
				queryParam.remark=remark.value;
			}
			if(radio2.value!="ASIN"){
				queryParam.isparent="isparent";
			}
			queryParam.disable=disabletype.value;
			queryParam.name=proname.value;
			refreshTable();
			isload=false;
		}
		function getOwner(id){
			queryParam.owner=id;
			if(isload==false){
				refreshTable();
			}
		}
		function getStatus(status){
			queryParam.salestatus=status;
			if(isload==false){
				refreshTable();
			}
		}
		function searchTypeChange(val){
			searchtype.value=val;
			queryParam.searchtype=val;
			if(isload==false){
				refreshTable();
			}
		}
		function disabletypeChange(val){
			disabletype.value=val;
			queryParam.disable=val;
			if(isload==false){
				refreshTable();
			}
		}
		function changeKeywords(val){
			searchKeywords.value=val;
			queryParam.search=val;
			if(isload==false){
				refreshTable();
			}
		}
		function searchConfirm(){
			refreshTable();
		}
		function getCategory(category){
			queryParam.category=category;
			moreSearchVisible.value=true;
		}
		function changeTag(tags){
			queryParam.taglist=tags;
		}
		function submitSearch(){
			queryParam.changeRate=salerangecheck.value;
			queryParam.isfba=isfba.value;
		    queryParam.remark=remark.value;
			queryParam.isbadreview=isbadreview.value;
			queryParam.name=proname.value;
			refreshTable();
			moreSearchVisible.value=false;
		}
		function resetSearch(){
			moreSearchVisible.value=false;
		}
		function refreshTable(){
			tableRef.value.loadData(queryParam);
		}
		function EditSaleFormula(){
			SaleFormulaRef.value.saleFormulaVisable = true
		}
	 
		function submitDataForm(){
			queryParam.paralist=columnval.value;
			searchConfirm();
			dataSearchVisible.value=false;
		}
		function resetDataForm(){
			dataSearchVisible.value=false;
		}
		function addColumn(){
			var data1="";var data2="";var data3=dataValue.value;
			if(dataColumn.value=="v.advacos"){
				data1="ACOS";
			}
			if(dataColumn.value=="v.advctr"){
				data1="CTR";
			}
			if(dataColumn.value=="v.acoas"){
				data1="ACOAS";
			}
			if(dataColumn.value=="v.advspc"){
				data1="CR";
			}
			if(dataColumn.value=="v.sessionrate"){
				data1="七日转化率";
			}
			if(dataColumn.value=="v.proprate"){
				data1="净利润率";
			}
			if(dataColumn.value=="v.dayfulfilla"){
				data1="可售天数";
			}
			if(dataColumn.value=="v.afn_fulfillable_quantity"){
				data1="库存";
			}
			if(dataColumn.value=="v.profitrate"){
				data1="产品利润率";
			}
			if(mark.value==3){
				data2=">";
			}
			if(mark.value==9){
				data2="=";
			}
			if(mark.value==6){
				data2="<";
			}
			columntext.value=columntext.value+(data1+data2+data3+";");
			columnval.value=columnval.value+(dataColumn.value+data2+data3+",");
		}
		function clearColumn(){
			columntext.value="";
			columnval.value="";
		}
		function showRefreshDialog(){
			if(selection.value&&selection.value.length>0){
				selection.value.forEach(function(item){
					 tableRef.value.refreshProduct(item);
				});
			}else{
				refreshRef.value.refreshProVisable=true;
			}
		}
		function refreshFBAInv(){
			//批量刷新fba库存 要锁定一个店铺 一个站点。。
			if(queryParam.groupid=="all" || queryParam.groupid=="" || queryParam.marketplaceid=="all" || queryParam.marketplaceid==""){
				ElMessage({
					 message: '批量刷新FBA库存需确定一个店铺下的某个站点！',
					 type: 'error'
				})
				return;
			}
			if(selection.value&&selection.value.length>0){
				var skus="";
				selection.value.forEach(function(item){
					skus=skus+item.sku+",";
				});
				skus=skus.substring(0,skus.length-1);
				inventoryRptApi.syncInventorySupply({"skus":skus,"groupid":queryParam.groupid,"marketplaceid":queryParam.marketplaceid}).then((res)=>{
					 if(res.data){
						 rows.afnFulfillableQuantity=res.data.afnFulfillableQuantity;
						 ElMessage({
							 message: '更新成功！',
							 type: 'success'
						})
						rows.itemshow=false;
					 }else{
						 ElMessage({
							  message: '更新失败！',
							  type: 'error'
						 })
					 }
				})
			}
			 
			
			
		}
			return {
				options,props,salerange,salerangecheck,
				radio2,modifyPrice,checkRows,selection,
				priceDialogRef,rankData,currentRank,rankChange,
				currentSolt,soltData,soltChange,getGroup,getOwner,getStatus,searchtype,
				searchTypeChange,isfba,saleslist,tableRef,refreshTable,changeKeywords,searchKeywords,
				searchConfirm,remark,submitSearch,resetSearch,moreSearchVisible,dataSearchVisible,
				SaleFormulaRef,EditSaleFormula,disabletype,disabletypeChange,
				isbadreview,proname,columnList,dataColumn,mark,dataValue,submitDataForm,resetDataForm,
				addColumn,clearColumn,columntext,columnval,showRefreshDialog,refreshRef,refreshFBAInv,
				changeTag,getCategory,
			}
		},
	}
</script>

<style>
	.r_active{
		background-color: var(--el-dropdown-menuItem-hover-fill);
	    color: var(--el-dropdown-menuItem-hover-color);
		}
	
</style>
