<template>
	<view>
		<view v-for="(wj,i) in wenjuanList" :key="i">
			<wenjuan class="test" :hasOther="wj.hasOther" :type="wj.type" :value="wj.value" :maxSelect="wj.maxSelect" :title="wj.title" :subTitle="wj.subTitle" :defaultValue="wj.defaultValue" @change="change($event,wj.id,wj.name)" @send="sendCode($event,wj.id,wj.name)"></wenjuan>
		</view>
		<view id="submitButton" @tap="submit">提交问卷(无后台)</view>
	</view>
</template>

<script>
	var CryptoJS = require("crypto-js");
	var prompt = ("入组标准：\n" + "1、60 years or more at time of transplantation." +
		"根据以上内容生成每个小点对应的英文问题，向患者提问，问题数量适中，必须要有判断题也要有填空题。直接生成问题,生成的问题不要重复，以json形式[{\"question\":\"\",\"type\":\"\",\"option\":[]}]输出。(注：填空题type为1，判断题type为2，question为问题，type是问题的类型，option为选项)英文输出，只输出JSON不要生成其他内容" +
		"例如：入组标准：1、60 years or more at time of transplantation. 2、Acute leukemia. 3、Myelodysplastic syndrome. 4、Myeloproliferative neoplasm.生成每个小点对应的问题，向患者提问，必须要有判断题也要有填空题。直接生成问题，以json形式{\"question\":\"\",\"type\":\"\",\"option\":[]}输出。(注：填空题type为1，判断题type为2，question为问题，type是问题的类型，option为选项)英文输出。" +
		"生成结果：[{\"question\":\"What is your age at the time of transplantation?\",\"type\":1,\"option\":\"\"},{\"question\":\"Do you have acute leukemia?\",\"type\":2,\"option\":[\"Yes\",\"No\"]},{\"question\":\"Have you been diagnosed with myelodysplastic syndrome?\",\"type\":2,\"option\":[\"Yes\",\"No\"]},{\"question\":\"Have you been diagnosed with myeloproliferative neoplasm?\",\"type\":2,\"option\":[\"Yes\",\"No\"]}]"
		)
	import wenjuan from '@/components/gwh-wenjuan/gwh-wenjuan.vue'
	export default {
		components:{
			wenjuan
		},
		onLoad() {
			// this.getQuestions()
		},
		data() {
			return {
				questionlist: [],
				wenjuanList:[
					{
						id:"1",
						name:"question1",
						type:"select",
						title:"Question 1",
						subTitle:"this is the test for question1 whose type is 'select'",
						value:[
							"choose A in question 1",
							"choose B in question 1",
							"choose C in question 1",
							"choose D in question 1"
						],
						defaultValue:"",
						hasOther:true,
						//用于接收已选择的数据
						resultValue:[]
					},
					{
						id:"2",
						name:"question2",
						type:"multi-select",
						title:"Question 2",
						subTitle:"this is the test for question 2 whose type is 'multi-select'",
						value:[
							"choose A in question 2",
							"choose B in question 2",
							"choose C in question 2",
							"choose D in question 2"
						],
						defaultValue:[
							"choose D in question 2",
							"choose A in question 2",
						],
						maxSelect:2,
						hasOther:false,
						//用于接收已选择的数据
						resultValue:[]
					},
					{
						id:"3",
						name:"question3",
						type:"boolean",
						title:"Question 3",
						subTitle:"this is the test for question 3 whose type is 'boolean'",
						value:[
							"是",
							"否"
						],
						defaultValue:"",
						
						//用于接收已选择的数据
						resultValue:[]
					},
					{
						id:"4",
						name:"question4",
						type:"level",
						title:"Question 4",
						subTitle:"this is the test for question 4 whose type is 'level'",
						value:[1,10,2],
						defaultValue:"2",
						
						//用于接收已选择的数据
						resultValue:[]
					},
					{
						id:"5",
						name:"question5",
						type:"input",
						title:"Question 5",
						subTitle:"this is the test for question 5 whose type is 'input'",
						maxSelect:10,
						defaultValue:"input charactors less than 10",
						
						//用于接收已选择的数据
						resultValue:[]
					},
					{
						id:"6",
						name:"question6",
						type:"textarea",
						title:"Question 6",
						subTitle:"this is the test for question 6 whose type is 'textarea'",
						maxSelect:200,
						defaultValue:"input charactors less than 200",
						
						//用于接收已选择的数据
						resultValue:[]
					},
					{
						id:"7",
						name:"question7",
						type:"date",
						title:"Question 7",
						defaultValue:"2020-05-08",
						subTitle:"this is the test for question 7 whose type is 'date'",
						
						//用于接收已选择的数据
						resultValue:[]
					},
					{
						id:"8",
						name:"question8",
						type:"time",
						title:"Question 8",
						defaultValue:"10:00",
						value:["09:00","12:00"],
						subTitle:"this is the test for question 8 whose type is 'time'",
						
						//用于接收已选择的数据
						resultValue:[]
					},
					{
						id:"9",
						name:"question9",
						type:"region",
						title:"Question 9",
						subTitle:"this is the test for question 9 whose type is 'region'",
						
						//用于接收已选择的数据
						resultValue:[]
					},
					{
						id:"10",
						name:"question10",
						type:"image",
						title:"Question 10",
						subTitle:"this is the test for question 10 whose type is 'image': every image could longtap to replace, delete or reorder, or to tap to view the whole image",
						maxSelect:3,
						
						//用于接收已选择的数据
						resultValue:[]
					},
					{
						id:"11",
						name:"question11",
						type:"phone",
						title:"Question 11",
						subTitle:"this is the test for question 11 whose type is 'phone'",
						value:[4,30],
						
						//用于接收已选择的数据
						resultValue:[]
					},
					{
						id:"12",
						name:"question12",
						type:"work",
						title:"Question 12",
						subTitle:"this is the test for question 12 whose type is 'work'",
						
						//用于接收已选择的数据
						resultValue:[]
					},
					{
						id:"13",
						name:"question13",
						type:"university",
						title:"Question 13",
						subTitle:"this is the test for question 13 whose type is 'university'",
						
						//用于接收已选择的数据
						resultValue:[]
					},
					{
						id:"14",
						name:"question14",
						type:"single-picker",
						title:"Question 14",
						subTitle:"this is the test for question 14 whose type is 'single-picker'",
						value:["choice 1","choice 2","choice 3"],
						defaultValue:"1",
						
						//用于接收已选择的数据
						resultValue:[]
					},
					{
						id:"15",
						name:"question15",
						type:"password",
						title:"Question 15",
						subTitle:"this is the test for question 14 whose type is 'password'",
						maxSelect:6,
						defaultValue:"请输入密码",
						value:["true"],
						
						//用于接收已选择的数据
						resultValue:[]
					},
					{
						id:"16",
						name:"question16",
						type:"foreign",
						title:"Question 16",
						subTitle:"this is the test for question 16 whose type is 'foreign'",
						
						//用于接收已选择的数据
						resultValue:[]
					},
					{
						id:"17",
						name:"question17",
						type:"local",
						title:"Question 17",
						subTitle:"this is the test for question 17 whose type is 'local'",
						
						//用于接收已选择的数据
						resultValue:[]
					},
					{
						id:"18",
						name:"question18",
						type:"drive",
						title:"Question 18",
						subTitle:"this is the test for question 17 whose type is 'drive'",
						
						//用于接收已选择的数据
						resultValue:[]
					},
				]
			}
		},
		methods: {
			change(e,...res){
				console.log(e);
				console.log(res);
				
				//用于更新数据
				for(var i=0;i<this.wenjuanList.length;i++){
					if(this.wenjuanList[i].id == res[0]){
						this.wenjuanList[i].resultValue = e;
					}
				}
			},
			sendCode(e,...res){
				console.log("please send check code to the phone:"+e[0])
				console.log(res);
			},
			submit(){
				for(var i=0;i<this.wenjuanList.length;i++){
					console.log("问题标号: "+this.wenjuanList[i].id);
					console.log("用户答案: ");
					console.log(this.wenjuanList[i].resultValue);
					
					//do something else
				}
			},
			getQuestions() {
				// console.log(generateToken("HS256"))
				uni.request({
					url: 'https://open.bigmodel.cn/api/paas/v4/chat/completions', //仅为示例，并非真实接口地址。
					method: "POST",
					data: {
						model: 'glm-3-turbo',
						messages: [{
							"role": "user",
							"content": prompt
						}]
					},
					header: {
						"Content-Type": "application/json", //自定义请求头信息
						"Authorization": generateToken()
					},
					success: (res) => {
						// console.log(res.data.choices[0].message);
						this.questionlist = res.data.choices[0].message.content
						console.log((this.questionlist))
						if (this.questionlist.includes('```json')) {
							var list = extractStringBetweenCharacters(this.questionlist, '```json', '```')[0]
			
							// this.text = 'request success';
							console.log(JSON.parse(list))
							this.wenjuanList = this.transferToQuestionList(JSON.parse(list))
						} else {
							var list = this.questionlist
							// console.log(JSON.parse(list))
							this.wenjuanList = this.transferToQuestionList(JSON.parse(list))
						}
			
					}
				});
			
			},
			transferToQuestionList(list) {
				var showList = []
				for (var i = 0; i < list.length; i++) {
					var item = {
						id: "",
						name: "",
						type: "",
						title: "",
						subTitle: "",
						value: [],
						defaultValue: "",
						hasOther: false,
						//用于接收已选择的数据
						resultValue: []
					}
					item.id = i
					item.subTitle = list[i].question
					if (list[i].type == 1) {
						item.type = "input"
						item.value = ''
						item.maxSelect = 50
					} else {
						item.type = "select"
						item.value = list[i].option
			
					}
			
					item.title = "Question" + i + ""
					item.hasOther = false
					item.name = "question" + i + ""
					showList.push(item)
				}
				console.log("用于显示的list", showList)
				return showList
			}
		}
		
	}
	
	function extractStringBetweenCharacters(str, startChar, endChar) {
		// 构建正则表达式，其中startChar和endChar是您想要提取的字符串的开始和结束字符
		const regex = new RegExp(startChar + '([^' + endChar + ']*)' + endChar, 'g');
	
		// 使用match函数查找所有匹配项
		const matches = str.match(regex);
		// 如果没有匹配项，返回null或空数组
		if (matches === null) {
			return null;
		}
		// 返回所有匹配的字符串数组
		return matches.map(match => match.slice(startChar.length, -endChar.length));
	}
	
	function base64UrlEncode(str) {
		var CryptoJS = require("crypto-js");
		var encodedSource = CryptoJS.enc.Base64.stringify(str);
		var reg = new RegExp('/', 'g');
		encodedSource = encodedSource.replace(/=+$/, '').replace(/\+/g, '-').replace(reg, '_');
		return encodedSource;
	}
	
	function generateToken() {
		var apikey = "358d7ae96eb4e3f0d5ad5d69ff4c9a2c.MqA4yp2RkA1rgWSh"
		let header = JSON.stringify({
			"alg": "HS256",
			"sign_type": "SIGN"
		})
		let secretSalt = apikey.split(".")[1]
		// console.log(id)
		// console.log(secretSalt[1])
		var iat = new Date().getTime()
		// console.log(new Date().getTime())
		var exp = iat + 12 * 60 * 60 * 1000
		// console.log(exp - iat);
		let payload = JSON.stringify({
			"api_key": apikey.split(".")[0],
			"exp": exp,
			"timestamp": new Date().getTime()
		})
	
		// let secretSalt = secretSalt[1];
		let before_sign = base64UrlEncode(CryptoJS.enc.Utf8.parse(header)) + '.' + base64UrlEncode(CryptoJS.enc.Utf8.parse(
			payload));
		let signature = CryptoJS.HmacSHA256(before_sign, secretSalt);
		signature = base64UrlEncode(signature);
	
		let final_sign = before_sign + '.' + signature;
		return final_sign
	}
</script>

<style>
	#submitButton{
		background-color: #3090d9;
		color: #FFFFFF;
		width: 710upx;
		padding: 10upx;
		margin-left: 10upx;
		text-align: center;
		margin-top: 15upx;
		margin-bottom: 15upx;
		border-radius: 10upx;
	}
</style>
