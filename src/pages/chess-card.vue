<template>
  <div>
    <hg-Header></hg-Header>
    <div class="chess-card-container">
      <div class="game_chesscard_box" :style="{height:windowHeight+'px'}">
        <div class="chesscard_main">
          <div class="chesscard_bg">
            <div class="chesscard-1"></div>
            <div class="chesscard2"></div>
            <div class="gold-coins"></div>
            <div class="Group-of-chesscard-1"></div>
            <div class="Group-of-chesscard-2"></div>
            <div class="logo"></div>
            <div class="SHARK"></div>
          </div>
          <div style="border-radius:48px;overflow:hidden" class="chesscard_main_bottom">
						<div class="chesscard_title">
							<p>请用彩票账户登录棋牌APP进行游戏</p>
						</div>
            <div class="chesscard_btn_bg">
              <div style="line-height:25px;">
                <div style="word-break: keep-all;">棋牌游戏余额：</div>
                <div class="chesscard_money_balance">{{whBalance}}</div>
              </div>

              <div class="money_transfer_btn" @click="transferMount">
                额度转换
              </div>
            </div>
						<div class="chesscard_qr_wrapper">
							<div class="chesscard_qr">
								<img :src="qrImg" alt="">
							</div>

							<div class="chesscard_btns">
								<div>
									<p>扫一扫</p>
									<p>下载移动应用</p>
								</div>
								<button @click="appHelpCode = 1">苹果APP教程</button><br>
								<button @click="appHelpCode = 2">安卓APP教程</button>
							</div>

							<app-help v-if="appHelpCode" :code.sync="appHelpCode"></app-help>
						</div>
            <!--<div>
              <div class="chesscard_intro_p4" @click="getPlayUrl(2)">
                &lt;!&ndash; <div class="try-to-play" @click="getPlayUrl(1)">

                  </div> &ndash;&gt;
                <div class="start-to-play">

                </div>
                <div class="breathing">
                  开始<br/> 游戏
                </div>
              </div>
            </div>-->

          </div>

        </div>
      </div>
    </div>
    <div class="overlay" v-if="isShowTransfer" @click="closeDialog"></div>
    <div class="ag-page-transfer" v-if="isShowTransfer">
      <div class="cha el-icon-circle-close-outline" @click="closeDialog"></div>
      <!--<transfer v-on:transferSuccess="transferSuccess" transferApi="/Wh_H5_Api/PrepareCreditOrder" title="棋牌"></transfer>-->
      <transfer v-on:transferSuccess="transferSuccess" transferApi="/Wh_APP_Api/PrepareCreditOrder" title="棋牌"></transfer>
    </div>
  </div>
</template>
<script>
import Transfer from "./Payment/transfer1";
import { Message } from "element-ui";
import HgHeader from "../components/hgHeader";
import AppHelp from '../components/appHelp'

export default {
  components: {
    Transfer,
    HgHeader,
		AppHelp
  },
  data() {
    return {
			qrImg: '',// 二维码图片地址
			appHelpCode: 0,// 0：关闭APP教程，1：打开苹果APP教程，2：打开安卓APP教程
      agBalance: "",
      isShowTransfer: false,
      windowHeight: "",
      tryPlayLink: "",
      realPlayLink: "",
      isTryPlayer: false,
      whBalance: "0.00"
    };
  },
  computed: {
    // 'windowHeight':this.$window&&(this.$window.innerHeight+'px')
  },
	created() {
		this.getQrImg()
	},
  mounted() {
    let _this = this;
    _this.windowHeight = this.$window.innerHeight;
    this.$window.onresize = function() {
      _this.windowHeight = _this.$window.innerHeight;
    };
    this.isTryPlayer = sessionStorage.getItem("im_realname") == "游客";
    if (!this.isTryPlayer) {
      this.getWhBalance();
    }
  },
  methods: {
		// 检测是否创建了棋牌账号,没创建的话就创建账号
		// 调用棋牌任何接口就会创建，不用再去判断和创建
		/*isCreatedAccount() {
			this.$http.post("Wh_APP_Api/is_open", JSON.stringify({oid: sessionStorage.getItem('im_token')})).then(res => {
				if (!res.data.data.whopen) {
					// 还没创建账号，开始创建
					this.createAccount()
				}
			})
		},*/
		// 创建账号
		/*createAccount() {
			this.$http.post("Wh_APP_Api/index", JSON.stringify({oid: sessionStorage.getItem('im_token')})).then(res => {
				if (!res.data.msg == '2006') {
					console.log('创建账号成功')
				} else {
					console.log('创建账号失败')
				}
			})
		},*/
		// 获取二维码图片
  	getQrImg() {
			this.$http.post("download/wh_app_down").then(res => {
				this.qrImg = res.data.data.wh_qrcode_url
			})
		},
    getPlayUrl(type) {
      if (type == 1) {

        let params = {};

        //试玩游戏链接
        this.$http
          .post("/aginfo/getAgGameLink/0/6", JSON.stringify(params))
          .then(res => {
            if (res.status === 200) {
              if (res.data.msg === 2006) {
                window.open(res.data.link);
                this.tryPlayLink = res.data.link;
                window.open(res.data.link);
                this.tryPlayLink = res.data.link;
              }
              if (res.data.msg === 7001) {
                Message({
                  type: "error",
                  showClose: true,
                  message: res.data.info
                });
              }
            }
          });
      }
      if (type == 2) {
        if (
          sessionStorage.getItem("im_username") === "游客" ||
          !sessionStorage.getItem("im_token")
        ) {
          this.$swal({
            text: "请登录正式会员账号后体验",
            type: "error",
            timer: 1200,
            confirmButtonText: "关闭"
          })
            .then(function(response) {})
            .catch(e => {});
          return;
        }
        let winHandler = window.open("", "_blank");

        let params = {};

        //真钱模式游戏链接
        this.$http
          // .post("/Wh_H5_Api/RedirectLogin", JSON.stringify(params))
          .post("/Wh_APP_Api/RedirectLogin", JSON.stringify(params))
          .then(res => {
            if (res.status === 200) {
              if (res.data.msg === 2006) {
                this.realPlayLink = res.data.link;
                winHandler.location.href = res.data.link;
              }
              if (res.data.msg === 7001) {
                Message({
                  type: "error",
                  showClose: true,
                  message: res.data.info
                });
                winHandler.close();
              }
            }
          });
      }
    },
		//获取棋牌游戏额度
		getWhBalance() {
      this.$http
        // .post("/Wh_H5_Api/getWhInfo", JSON.stringify({oid: sessionStorage.getItem("im_token")}))
        .post("/Wh_APP_Api/getWhInfo", JSON.stringify({oid: sessionStorage.getItem("im_token")}))
        .then(res => {
          if (res.status === 200) {
            if (res.data.msg === 2006) {
              this.whBalance =
                parseInt(res.data.balance.whBalance).toFixed(2) || "0.00";
              this.$store.commit(
                "updatelotteryMoney",
                res.data.balance.userBalance
              );
            }
          }
        });
    },
    transferSuccess(msg) {
      this.getWhBalance();
      this.closeDialog();
    },
    closeDialog() {
      this.isShowTransfer = false;
    },
    transferMount() {
      if (
        sessionStorage.getItem("im_username") === "游客" ||
        !sessionStorage.getItem("im_token")
      ) {
        this.$swal({
          text: "请登录正式会员账号后体验",
          type: "error",
          timer: 1200,
          confirmButtonText: "关闭"
        })
          .then(function(response) {})
          .catch(e => {});
        return;
      }
      this.isShowTransfer = true;
    }
  }
};
</script>
<style>
.ag-page-transfer {
  z-index: 99;
  width: 340px;
  position: absolute;
  top: 20%;
  left: 50%;
  margin-left: -170px;
  background-color: #fff;
  border-radius: 6px;
  color: #333;
}
.ag-page-transfer .grid-content .conmit div {
  width: 145px;
}
.ag-page-transfer .el-main {
  overflow: hidden;
}
.ag-page-transfer .el-main .el-row {
  margin-top: 15px;
}
.ag-page-transfer .grid-content .conmit {
  border: none;
  padding: 15px 0;
}
.ag-page-transfer .foot,
.ag-page-transfer .huan {
  display: none;
}
.ag-page-transfer .el-col-3 {
  width: auto;
}
.ag-page-transfer .title {
  background-color: #e1e2ee;
  height: 33px;
  line-height: 33px;
  text-align: left;
  padding: 0 20px;
  border-top-left-radius: 8px;
  border-top-right-radius: 8px;
  font-size: 16px;
  color: #666;
  font-weight: normal;
}
.ag-page-transfer .cha {
  font-size: 30px;
  right: -30px;
  top: -30px;
  height: 30px;
  width: 30px;
}
.ag-page-transfer .grid-content .conmit div {
  height: 30px;
  line-height: 30px;
}
.ag-page-transfer .grid-content input {
  width: 190px;
  border-radius: 4px;
}
</style>

<style scoped>
.chess-card-container {
  width: 100%;
  height: 100%;
  background: url("../../static/images/common/chesscard/background1.jpg");
  background-color: black;
}
@keyframes breathing {
  0% {
    box-shadow: 0 0 0px 0px #5fe8ef;
  }
  100% {
    box-shadow: 0 0 20px 20px #befafd;
  }
}

.breathing {
  -webkit-animation: breathing 2s ease-out infinite alternate-reverse;
  animation: breathing 2s ease-out infinite alternate-reverse;
}

.chesscard_amount_title {
  font-size: 14px;
}
.ag-page-transfer {
  z-index: 99;
  width: 340px;
  position: absolute;
  top: 20%;
  left: 50%;
  margin-left: -170px;
  background-color: #fff;
  border-radius: 6px;
  color: #333;
}
.ag-page-transfer .grid-content .conmit div {
  width: 145px;
}
.ag-page-transfer .el-main {
  overflow: hidden;
}
.ag-page-transfer .el-main .el-row {
  margin-top: 15px;
}
.ag-page-transfer .grid-content .conmit {
  border: none;
  padding: 15px 0;
}
.ag-page-transfer .foot,
.ag-page-transfer .huan {
  display: none;
}
.ag-page-transfer .el-col-3 {
  width: auto;
}
.ag-page-transfer .title {
  background-color: #e1e2ee;
  height: 33px;
  line-height: 33px;
  text-align: left;
  padding: 0 20px;
  border-top-left-radius: 8px;
  border-top-right-radius: 8px;
  font-size: 16px;
  color: #666;
  font-weight: normal;
}
.ag-page-transfer .cha {
  font-size: 30px;
  right: -30px;
  top: -30px;
  height: 30px;
  width: 30px;
}
.ag-page-transfer .grid-content .conmit div {
  height: 30px;
  line-height: 30px;
}
.ag-page-transfer .grid-content input {
  width: 190px;
  border-radius: 4px;
}
</style>

<style lang="less" rel="stylesheet/less" scoped>
.chesscard_billion_money {
  font-size: 50px;
  color: #feef00;
  font-family: "Alfa Slab One";
  font-weight: bold;
}
.chesscard_game_jiangli {
  color: #00fce9;
  font-weight: bold;
}
.chesscard_intro_p1 {
  font-size: 20px;
  margin-top: 36px;
}

.chesscard_intro_p2 {
  font-size: 20px;
}

.chesscard_intro_p3 {
  margin-top: 36px;
  font-weight: bold;
}

.chesscard_intro_p4 {
  cursor: pointer;
  position: relative;
}

.chesscard_intro_p4 > div:last-child {
  position: absolute;
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fcfbdd;
  border-radius: 100px;
  box-shadow: 0 0 20px 10px #5fe8ef;
}

.try-to-play {
  width: 230px;
  background: url("/static/images/common/chesscard/try-button.png") no-repeat;
  margin-right: 10px;
  transition: all linear 0.2s;
}

.start-to-play {
  width: 182px;
  height: 184px;
  background: url("/static/images/common/chesscard/start-button1.png") no-repeat;
  background-position: center -72px;
  transition: transform ease-in-out 0.3s;
}

.chesscard_intro_p4:hover .start-to-play {
  transform: rotate(120deg);
}

.game_chesscard_box {
  background-color: black;
  background: url("/static/images/common/chesscard/background.png") no-repeat
    center center;
  display: flex;
  background-size: cover;
  align-items: flex-start;
  justify-content: center;
  color: white;
  font-family: "Microsoft Yahei";
  position: relative;
}

.chesscard_main_bottom {
  display: flex;
  /*justify-content: center;*/
  flex-direction: column;
  flex: 1;
  align-items: center;
  /*justify-content: space-around;*/
  min-height: 300px;
}

.chesscard_title {
	font-size: 20px;
	color: #E5FBFB;
	margin: 30px 0 16px;
	letter-spacing: 0.6px;
	line-height: 1.4;
	text-shadow: 2px 2px 5px #222;
	font-weight: 500;
	font-size: 22px;
	strong{
		font-size: 36px;
	}
	span{
		font-size: 20px;
	}
}

.chesscard_qr_wrapper{
	display: flex;
	margin-top: 20px;
}

.chesscard_qr{
	width: 130px;
	height: 130px;
	padding: 1px 0 0 1px;
	background-color: white;
}

.chesscard_qr > img{
	width: 100%;
	height: 100%;
}

.chesscard_btns{
	line-height: 1.5;
	margin-left: 20px;
}

.chesscard_btns > div{
	margin-bottom: 4px;
}

.chesscard_btns button{
	margin-top: 8px;
	padding: 6px 24px;
	color: #51C8F1;
	background: none;
	border: 1px solid #51C8F1;
	border-radius: 4px;
	cursor: pointer;
}

.chesscard_btns button:hover{
	color: white;
	background-color: #2E92FF;
	border-color: #2E92FF;
}

/*.chesscard_main_bottom > div:last-child {
  font-family: "Microsoft Yahei";
  font-weight: bold;
  font-size: 35px;
  width: 100%;
  display: flex;
  justify-content: center;
}*/

.chesscard_main {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}
.chesscard_main > div {
  width: 100%;
  left: 0;
}

.chesscard_main > div:first-child {
  top: 0;
  z-index: 5;
  height: 50%;
}

.game_chesscard_content {
  position: absolute;
  width: 68%;
  height: 32%;
  bottom: 16%;
  left: 16%;
  border: solid thin white;
  color: white;
}
.chesscard_btn_bg {
  width: 384px;
  background-color: #12588f;
  border-radius: 10px;
  display: inline-flex;
  justify-content: space-between;
  align-items: center;
  box-sizing: border-box;
  padding: 16px;
  flex-wrap: nowrap;
}
.chesscard_money_balance {
  display: inline-block;
  color: #ffe440;
  font-size: 20px;
  padding-right: 10px;
  font-weight: bold;
}
.money_transfer_btn {
  width: 140px;
  height: 47px;
  background-image: url("/static/images/common/chesscard/transfer-button.png");
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  border-radius: 8px;
  background-size: 100% 100%;
  background-repeat: no-repeat;
  background-color: #56d275;
}
.money_transfer_btn:hover {
  background-image: none;
}

.chesscard_bg {
  position: relative;
}

.chesscard_bg > * {
  display: none;
}

.chesscard_bg > div {
  background-repeat: no-repeat;
  background-position: center center;
  position: absolute;
}
</style>
