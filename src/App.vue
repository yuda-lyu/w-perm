<template>
    <div
        v-domresize
        @domresize="resize"
    >

        <LayoutState :style="`opacity:${ready?0:1};`" v-if="!ready"></LayoutState>

        <transition enter-active-class="fade-enter-active" leave-active-class="fade-leave-active">
            <Layout v-if="ready"></Layout>
        </transition>

        <LoadingWinBar></LoadingWinBar>
        <CheckYesNo></CheckYesNo>
        <CheckYes></CheckYes>

        <VeCrules></VeCrules>
        <VeCpemis></VeCpemis>
        <VeCgrups></VeCgrups>

        <VeGrupBlngUsers></VeGrupBlngUsers>
        <VePemiBlngGrups></VePemiBlngGrups>

    </div>
</template>

<script>
import get from 'lodash-es/get.js'
import cloneDeep from 'lodash-es/cloneDeep.js'
import isestr from 'wsemi/src/isestr.mjs'
import iseobj from 'wsemi/src/iseobj.mjs'
import isDev from 'wsemi/src/isDev.mjs'
import wui from 'w-ui-loginout/src/WUiLoginout.mjs'
import LoadingWinBar from './components/Common/LoadingWinBar.vue'
import CheckYesNo from './components/Common/CheckYesNo.vue'
import CheckYes from './components/Common/CheckYes.vue'
import LayoutState from './components/LayoutState.vue'
import Layout from './components/Layout.vue'
import VeCrules from './components/VeCrules.vue'
import VeCpemis from './components/VeCpemis.vue'
import VeCgrups from './components/VeCgrups.vue'
import VeGrupBlngUsers from './components/VeGrupBlngUsers.vue'
import VePemiBlngGrups from './components/VePemiBlngGrups.vue'


export default {
    components: {
        LoadingWinBar,
        CheckYesNo,
        CheckYes,
        LayoutState,
        Layout,
        VeCrules,
        VeCpemis,
        VeCgrups,
        VeGrupBlngUsers,
        VePemiBlngGrups,
    },
    data: function() {
        return {
            ll: null,
        }
    },
    beforeMount: function() {
        // console.log('methods beforeMount')

        let vo = this

        //setVo, 更換ui內vo, 才能使用廣播技術, 更換語系才能用廣播通知全部組件forceUpdate
        vo.$ui.setVo(vo)

        //setLang
        let lang = get(window, '___pmwperm___.language', '')
        vo.$ui.setLang(lang, 'app init') //初始化先讀取html內語系設定進行變更
        // console.log('lang', lang)

        function loginSuccess(data) {
            console.log('login success', cloneDeep(data.user))
            vo.$ui.updateConnState('csLogin')
            vo.$ui.updateUserToken(data.token)
            vo.$ui.updateUserSelf(data.user)
        }

        function loginError(data) {
            console.log('login error', cloneDeep(data))
            vo.$ui.updateConnState('csErrLogin')
            vo.$ui.updateUserToken('')
            let urlRedirect = get(window, '___pmwperm___.urlRedirect', '')
            if (!isestr(urlRedirect)) {
                console.log('urlRedirect', urlRedirect)
                throw new Error(`invalid urlRedirect`)
            }
            if (isDev()) {
                console.log('60s redirect to:', urlRedirect)
                setTimeout(() => {
                    window.location.href = urlRedirect
                }, 60 * 1000)
            }
            else {
                window.location.href = urlRedirect
            }
        }

        //login
        console.log('login...')
        let ll = wui('wperm', {
            timeWaitAnimation: 2000,
            params: {},
        })
        ll.login({
            afterGetUser: null,
            afterLogin: null,
            loginSuccess,
            loginError,
        })
        vo.ll = ll

    },
    mounted: function() {
        let vo = this

        //set, 把目前vo儲存至window供外部非vue環境使用
        window.$vo = vo

    },
    computed: {

        ready: function() {
            //console.log('computed ready')

            let vo = this

            let connState = get(vo, `$store.state.connState`)
            let webInfor = get(vo, `$store.state.webInfor`)

            let b1 = connState === 'csLogin'
            let b2 = iseobj(webInfor)
            let b = b1 && b2

            return b
        },

    },
    methods: {

        resize: function(msg) {
            // console.log('methods resize', msg)

            let vo = this

            //syncHeight
            vo.$ui.syncHeight()

        },

    },
}
</script>

<style>
html,
body {
    font-family: '微軟正黑體', 'Microsoft JhengHei', 'MicrosoftJhengHeiRegular', 'Avenir', Helvetica, Arial, sans-serif;
    overflow-y: hidden;
}

div,
p,
span,
a,
pre,
input,
textarea,
button {
    font-family: inherit;
}

.fade-enter-active {
  animation: go 1s;
}

.fade-leave-active {
  animation: back 1s;
}

@keyframes go {
  from { opacity: 0; }
  to {opacity: 1;}
}

@keyframes back {
  from { opacity: 1; }
  to { opacity: 0; }
}

</style>
