<template>
	<view>
		<text>1、60 years or more at time of transplantation.\n 2、Acute leukemia. \n3、Myelodysplastic syndrome or Myeloproliferative neoplasm.</text>
		<button @click="getQuestions()">点击生成问卷</button>
		<view v-for="(wj,i) in wenjuanList" :key="i">
			<wenjuan class="test" :hasOther="wj.hasOther" :type="wj.type" :value="wj.value" :maxSelect="wj.maxSelect"
				:title="wj.title" :subTitle="wj.subTitle" :defaultValue="wj.defaultValue"
				@change="change($event,wj.id,wj.name)" @send="sendCode($event,wj.id,wj.name)"></wenjuan>
		</view>
		<view id="submitButton" @tap="submit">提交问卷</view>
		<text>{{finalResult}}</text>
	</view>
	
</template>


<script>
	import wenjuan from '@/components/gwh-wenjuan/gwh-wenjuan.vue'
	var CryptoJS = require("crypto-js");
	var prompt = ("入组标准：\n" + "1、60 years or more at time of transplantation.2、Acute leukemia. 3、Myelodysplastic syndrome 4、Myeloproliferative neoplasm." +
		"根据以上内容生成每个小点对应的英文问题，向患者提问，问题数量适中，必须要有判断题也要有填空题。直接生成问题,生成的问题不要重复，以json形式[{\"question\":\"\",\"type\":\"\",\"option\":[]}]输出。(注：填空题type为1，判断题type为2，question为问题，type是问题的类型，option为选项)英文输出，只输出JSON不要生成其他内容" +
		"例如：入组标准：1、60 years or more at time of transplantation. 2、Acute leukemia. 3、Myelodysplastic syndrome. 4、Myeloproliferative neoplasm.生成每个小点对应的问题，向患者提问，必须要有判断题也要有填空题。直接生成问题，以json形式{\"question\":\"\",\"type\":\"\",\"option\":[]}输出。(注：填空题type为1，判断题type为2，question为问题，type是问题的类型，option为选项)英文输出。" +
		"生成结果：[{\"question\":\"What is your age at the time of transplantation?\",\"type\":1,\"option\":\"\"},{\"question\":\"Do you have acute leukemia?\",\"type\":2,\"option\":[\"Yes\",\"No\"]},{\"question\":\"Have you been diagnosed with myelodysplastic syndrome?\",\"type\":2,\"option\":[\"Yes\",\"No\"]},{\"question\":\"Have you been diagnosed with myeloproliferative neoplasm?\",\"type\":2,\"option\":[\"Yes\",\"No\"]}]"
	)
	// var questionlist = {}

	export default {
		components: {
			wenjuan
		},
		data() {
			return {
				// title: 'Hello',
				questionlist: [],
				wenjuanList: [],
				finalResult:''
			}
		},
		onLoad() {
			// this.getQuestions()
		},
		methods: {
			change(e, ...res) {
				console.log(e);
				console.log(res);

				//用于更新数据
				for (var i = 0; i < this.wenjuanList.length; i++) {
					if (this.wenjuanList[i].id == res[0]) {
						this.wenjuanList[i].resultValue = e;
					}
				}
			},
			sendCode(e, ...res) {
				console.log("please send check code to the phone:" + e[0])
				console.log(res);
			},
			submit() {
				var allresult = ''
				for (var i = 0; i < this.wenjuanList.length; i++) {
					// console.log("问题标号: " + this.wenjuanList[i].id);
					// console.log("用户答案: ");
					// console.log("问题:"+this.wenjuanList[i].subTitle+',患者回答:'+this.wenjuanList[i].resultValue);
					allresult = allresult + "问题:" + this.wenjuanList[i].subTitle + ',患者回答:' + this.wenjuanList[i]
						.resultValue + '\n'
					//do something else
				}
				console.log("最终答案: ", allresult)
				this.evalPatient(allresult)
			},
			evalPatient(result) {
				// console.log(generateToken("HS256"))
				uni.request({
					url: 'https://open.bigmodel.cn/api/paas/v4/chat/completions', //仅为示例，并非真实接口地址。
					method: "POST",
					data: {
						model: 'glm-4-turbo',
						messages: [{
							"role": "user",
							"content": ("入组标准:" +
								"1、60 years or more at time of transplantation. 2、Acute leukemia. 3、Simultaneous presence of Myelodysplastic syndrome and Myeloproliferative neoplasm." +
								"患者对问题的回答:" + result +
								"\n根据入组标准与患者的回答判断患者是否符合以上标准，并说明理由。患者必须同时符合所有的入组标准，否则即为不符合入组标准。只输出JSON," +
								"不要输出其他内容。注：result为判断结果（只回答Yes与No），" +
								"reason为所有不符合的标准的原因，若符合则reason输出为空,英文输出")
						}]
					},
					header: {
						"Content-Type": "application/json", //自定义请求头信息
						"Authorization": generateToken()
					},
					success: (res) => {
						// console.log(res.data.choices[0].message);
						this.finalResult = res.data.choices[0].message.content
						console.log((this.finalResult))
						

					}
				});

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
					item.id = '' + i + ''
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
		var apikey = "your key"
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
	//和普通base64加密不一样

	//final_sign:eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoic2NhciIsInJvbGUiOiJhZG1pbiIsImV4cGlyYXRpb25EYXRhIjoiMjAxOC0xMC0yNCAxNzowNTowMCJ9.bVc48JsxiM7ZZgtZch1QnRpLyt08M9LepwoLvs_aejI
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}

	#submitButton {
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