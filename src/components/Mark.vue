<script setup lang="ts">
import {ref,type Ref,watch} from 'vue'
import { ElMessageBox } from 'element-plus'
import MyUI from "./My.vue"
const sortBy=ref('allUsed')
const isLoading=ref(false)
const isShowMe=ref(false)
const showSimple=ref(true)
window.onresize=()=>{
  if (window.outerWidth>500)showSimple.value=false
  else showSimple.value=true
}
if (window.outerWidth>500)showSimple.value=false
else showSimple.value=true
const past=ref(false)
const grade=ref('3')
const dataText=ref("总流量")
const getDataText=()=>{
  let ret={allUsed:"总流量",averageSpeed:"平均速度",onlineTime:"在线时长"}[sortBy.value]
  return ret?ret:""
}
const props = defineProps({
  show: Object,
  loginInfo:{ type: Object, required: true },
})
const show=ref(false)
if(props.show){
    watch(props.show,(ns,os)=>{
            show.value=ns.show
    })
}
watch(show,(ns,os)=>{
    if(props.show)props.show.show=ns;
    if(ns)refreshMark()

})
const mark:Ref<any>=ref([])
const api =async(args:string[][])=>{
  args.push(["cache", window.location.host])
  const response = await fetch(import.meta.env.VITE_API_URL+"get.ajax?"+new URLSearchParams(args).toString(), {
    mode: "cors",
    redirect: "follow",
    referrerPolicy: "no-referrer"
  })
  const resp=await response.json()
  return resp
}

const typeMatch=(str:string)=>{
    const arr1=['移动','联通','电信','广电']
    const arr2=['','success','warning','danger']
    for(let i in arr1)if(str.includes(arr1[i]))return arr2[i]
    return 'info'
}

const refreshMark=async()=>{
  dataText.value=getDataText()
  isLoading.value=true
  mark.value=[]
  try{
    let rep=await api([["grade",grade.value],['sorted_by',sortBy.value],["isPast",past.value?"true":"false"]])
    rep.data.forEach((element:any) => {
      let formatted
      if(sortBy.value=='allUsed'){
        formatted=formatter(element.data, ['B', 'KB', 'MB', 'GB', 'TB', 'PB'], [0, 0, 0, 0, 1, 1])
      }else if(sortBy.value=='averageSpeed'){
        formatted=formatter(element.data * 8, ['Bps', 'Kbps', 'Mbps', 'Gbps', 'Tbps', 'Pbps'], [0, 0, 0, 1, 1, 1])
      }else if(sortBy.value=='onlineTime'){
        formatted=timeformatter(element.data)
      }
      element.data=formatted
      element.type=typeMatch(element.isp)
      mark.value.push(element)
    });
  }catch(err){
    ElMessageBox.alert('无法获取榜单信息，可能是后端服务器异常', '错误', {
      confirmButtonText: '确定'
    })
  }
  isLoading.value=false
}
function formatter(num: number, des: Array<string>, flo: Array<number>) {
  var cnum = num;
  var total_index = 0;
  while (cnum >= 1024) {
    if (total_index == des.length - 1) break;
    cnum = cnum / 1024;
    total_index++;
  }
  return cnum.toFixed(flo[total_index]) + des[total_index];
}
const timeformatter=(n:number)=>{
  if(n<60)return n.toFixed(0)+'秒'
  n/=60
  if(n<60)return n.toFixed(0)+'分钟'
  n/=60
  if(n<24)return n.toFixed(0)+'小时'
  n/=24
  return n.toFixed(0)+'天'
}
</script>

<style scoped>
.tag-short{
  max-width: 50px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.tag-long{
  max-width: 80px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
</style>