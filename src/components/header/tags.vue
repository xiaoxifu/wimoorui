<template>
		   <el-cascader 
		   v-model="tagsValue" 
		   @change="changeTags" 
		   :options="tagsList"  
		   :teleported="false" 
		   placeholder="请选择标签"  
		   :props="markprops" 
		   clearable />
</template>

<script setup>
	import { ref,reactive,onMounted,toRefs,defineEmits,defineExpose } from 'vue'
	import {getAllTags} from '@/api/sys/admin/tag.js';
	const emit = defineEmits(['change']);
	let state =reactive({
		tagsList:[],
		markprops:{ multiple: true },
		tagsValue:'',
	})
	let{ tagsList,tagsValue,markprops}=toRefs(state);
	onMounted(()=>{
		getAllTags().then(res => {
		    state.tagsList=res.data;
		});
	});
	
	function changeTags(tags){
		 var arrs=[];
		 if(tags){
		 	 tags.forEach(function(item){
		 		 arrs.push(item[1].toString())
		 	 });
		 }
		 emit("change",arrs);
	}
</script>

<style>
</style>