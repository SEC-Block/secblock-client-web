<template>
  <el-container>
    <!-- 白色大背景 -->
    <main>
      <!-- 内容展示区域 -->
      <!-- 创建新钱包 -->
      <el-row>
        <el-col :xs="2" :sm="0" :md="0">&nbsp;</el-col>
        <el-col :xs="20" :sm="24" :md="24">
          <section class="content_list">
            <section v-if="pages==1">
              <h2>{{$t('newWallet.createWallet')}}</h2>
              <public-ipt
                  v-model="walletPass1"
                  maxlength="30"
                  :class="formatError?'errorBorder':''" 
                  :placeholder="$t('newWallet.enterPass1')"
                  @input="inputContent1"
                  @loseFocus="loseFocus"
                  @getFocus="getFocus"
                  :value="walletPass1" />
              <tips-content v-show="showFormat" :tipsTxt="tipsTxt1"></tips-content>
              <public-ipt 
                  v-model="walletPass2"
                  maxlength="30"
                  class="mt1"
                  :placeholder="$t('newWallet.enterPass2')" 
                  :class="DontAgree?'errorBorder':''" 
                  @input="inputContent2"/>
              <tips-content v-show="errorTxt2" :tipsTxt="tipsTxt2"></tips-content>
              <tips-list :tipsListKey="tipsListPass" class="tips_m2"></tips-list>
              <public-btn type="button" @click.native="createWallet" :disabled="!createActive" :class="createActive?'btn_active':''">{{$t('newWallet.createWallet')}}</public-btn>
            </section>
          </section>

          <section class="create_cnt">
            <!-- 保存Keystore文件 -->
            <section v-if="pages==2">
              <h2>{{$t('newWallet.saveKeyStore')}}</h2>
              <span @click="downUrlTxt" class="down_keystore">{{$t('newWallet.keyStoreDown')}}</span>
              <tips-list :tipsListKey="tipsListKey" class="tips_m2"></tips-list>
              <public-btn type="button" class="continue_btn btn_active" @click.native="continueKey">{{$t('newWallet.next')}}</public-btn>
            </section>

            <!-- 保存私钥 -->
            <section v-if="pages==3">
              <h2>{{$t('newWallet.savePrivateKey')}}</h2>
              <article>
                {{ privateKey }}
              </article>
              <tips-list :tipsListKey="tipsListKey" class="tips_m3"></tips-list>
            </section>
            <!-- 内容展示区域结束 -->
          </section>  
        </el-col>
      </el-row>
      <!-- 公共背景底部 -->
      <content-footer></content-footer>
    </main>
  </el-container>
</template>

<script>
  import publicIpt from '../componentsPublic/pubic-ipt-pass'
  import publicBtn from '../componentsPublic/public-btn'
  import tipsContent from '../componentsPublic/tips-content'
  import contentFooter from '../componentsPublic/content-footer'
  import tipsList from './compoents/tips-list'
  import tipsImg from '../../assets/images/tipsImg.png'
  import walletsHandler from '../../lib/WalletsHandler.js'
  import walletMethods from '../../utils/publicMethode.js'
  const SECUtil = require('@sec-block/secjs-util')
  const CryptoJS = require('crypto-js')
  export default {
    name: 'createWallet',
    data() {
      return {
        walletPass1: '',
        walletPass2: '',
        errorTxt1: false, //密码输入框开始输入的时候显示
        errorTxt2: false, //两次密码不一致设置true
        formatError: false, //密码格式错误
        DontAgree: false, //两次密码不一致border
        showFormat: false,//密码提示不显示
        pages: 1, //控制显示页面 1 创建钱包  2 保存Keystore文件 3 保存私钥
        tipsTxt1: 'passTips.passFormat',
        tipsTxt2: 'passTips.passNoMatch',
        privateKey: '', //私钥
        userAddress: '', //地址
        privateKeyArr: [],
        tipsListPass: [{
            'id': '01',
            'tips_img': tipsImg,
            'tips_cnt': 'tipsListPass.tipsCnt1',
          },
          {
            'id': '02',
            'tips_img': tipsImg,
            'tips_cnt': 'tipsListPass.tipsCnt2',
          },
          {
            'id': '03',
            'tips_img': tipsImg,
            'tips_cnt': 'tipsListPass.tipsCnt3',
          }
        ],
        tipsListKey: [{
            'id': '01',
            'tips_img': tipsImg,
            'tips_cnt': 'tipsListKey.tipsCnt1',
          },
          {
            'id': '02',
            'tips_img': tipsImg,
            'tips_cnt': 'tipsListKey.tipsCnt2',
          },
          {
            'id': '03',
            'tips_img': tipsImg,
            'tips_cnt': 'tipsListKey.tipsCnt3',
          }
        ],
        keyData: '' //当前创建钱包的加密信息
      }
    },

    created() {
      
    },

    methods: {
      //失去焦点
      loseFocus () {
        this.showFormat = false
      },
      //得到焦点
      getFocus () {
        this.showFormat = true
      },
      //去掉开头空格
      inputContent1 (e) {
        this.walletPass1 = this.walletPass1.replace(/^\s+|\s+$/g, '')
        this.$nextTick(()=> {
          this.walletPass1 = this.walletPass1.replace(/[\u4E00-\u9FA5]/g,'')
        })
      },
      //去掉开头空格 不能输入中文
      inputContent2 () {
        this.walletPass2 = this.walletPass2.replace(/^\s+|\s+$/g, '')
        this.$nextTick(()=> {
          this.walletPass2 = this.walletPass2.replace(/[\u4E00-\u9FA5]/g,'')
        })
      },
      //创建钱包
      createWallet() {
        let pass1 = this.walletPass1.replace(/^\s+|\s+$/g, '')
        let pass2 = this.walletPass2.replace(/^\s+|\s+$/g, '')
        if (pass1 != pass2) {
          this.errorTxt2 = true
          this.DontAgree = true
          this.formatError = false
          this.tipsTxt2 = 'passTips.passNoMatch'
          return
        } else {
          this.errorTxt2 = false
          this.DontAgree = false
          this.formatError = false
          this.pages = 2 //保存Keystore文件
          
          let keys = SECUtil.generateSecKeys() //创建钱包
          let privKey64 = keys.privKey //获取创建钱包的私钥
          this.privateKey = keys.privKey //赋值当前显示私钥
          let privateKey = privKey64
          let englishWords = SECUtil.entropyToMnemonic(privKey64)
          let pubKey128 = keys.publicKey //公钥
          let pubKey128ToString = pubKey128.toString('hex')
          let userAddressToString = keys.secAddress //地址
          this.userAddress = keys.secAddress //赋值当前地址用作创建json文件
          
          let walletName = 'SEC' + ""+ keys.userAddress +"" //下载json文件名称
          let keyFileDataJS = {
            [walletName]: {
              'privateKey': privateKey,
              'publicKey': pubKey128ToString,
              'walletAddress': userAddressToString
            }
          }
          //通过密码加密钱包  
          let cipherKeyData = CryptoJS.AES.encrypt(JSON.stringify(keyFileDataJS), pass1)
          this.keyData = cipherKeyData.toString()
        }
      },
      //保存keyStore之后继续查看私钥
      continueKey() {
        this.pages = 3
      },
      //下载json文件
      downUrlTxt () {
        var filename = "SEC" + this.userAddress + ".json";
        this.funDownload(filename, "{'version':3,'data':'"+this.keyData+"'}");
      },
      // 下载文件方法
      funDownload (filename, content) {
          var eleLink = document.createElement('a');
          eleLink.download = filename;
          eleLink.style.display = 'none';
          // 字符内容转变成blob地址
          var blob = new Blob([content], {type: "application/octet-stream"});
          eleLink.href = URL.createObjectURL(blob);
          // 触发点击
          document.body.appendChild(eleLink);
          eleLink.click();
          // 然后移除
          document.body.removeChild(eleLink);
      }
    },
    components: {
      publicBtn,

      publicIpt,

      tipsContent,

      tipsList,

      contentFooter
    },
    computed: {
      //创建钱包按钮是否可点击
      createActive() {
        var pass = /^(?![\d]+$)(?![a-zA-Z]+$)(?![^\da-zA-Z]+$).{8,30}$/
        if (this.walletPass1 == this.walletPass2 && pass.test(this.walletPass2)) {
          this.errorTxt2 = false
          this.DontAgree = false
        }
        return this.walletPass1.length > 7 
          && pass.test(this.walletPass2)
          && pass.test(this.walletPass1) ? true : false
      }
    }
  }

</script>

<style scoped>
  main .el-row  {height: 30.35rem;}
  /* 保存keyStore */
  main .down_keystore {display: block;height: 2.4rem;background: rgba(255, 255, 255, 1);border-radius: .5rem;
    border: .05rem solid rgba(145, 162, 170, 1);text-align: center;line-height: 2.4rem;color: #42535B;font-size: .8rem;
    font-weight: 500;text-decoration: none;}
  main .down_keystore:hover {cursor: pointer;}
  main .continue_btn {width: 9.1rem;}
  /* 保存私钥 */
  main article {background: rgba(245, 245, 245, 1);color: #42535B;border: .05rem solid rgba(145, 162, 170, 1);
    border-radius: .5rem;font-size: .8rem;font-weight: 500;padding: 1rem;word-wrap: break-word;line-height: 1.5;}
  main .tips_m1 {margin: .4rem auto 1.8rem;}
  main .tips_m2 {margin: .8rem auto 2.2rem;}
  main .tips_m3 {margin: 1.2rem auto 0;}
</style>
