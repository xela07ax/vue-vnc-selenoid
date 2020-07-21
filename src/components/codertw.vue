<template>
    <div class="page-home" ref="canvas">
        <canvas id="noVNC_canvas" width="800" height="600">
            Canvas not supported.
        </canvas>
    </div>
</template>

<script>
    import WebUtil from '../../lib/noVNC/app/webutil.js'

    //import Base64 from '../../lib/noVNC/core/base64.js'
    //import Websock from '../../lib/noVNC/core/websock.js'
    import '../../lib/noVNC/core/des.js'
    import '../../lib/noVNC/core/input/keysymdef.js'
    import '../../lib/noVNC/core/input/xtscancodes.js'
    import '../../lib/noVNC/core/input/util.js'
    //import {Keyboard, Mouse} from '../../lib/noVNC/core/input/devices.js'
    //import Display from '../../lib/noVNC/core/display.js'
    import '../../lib/noVNC/core/inflator.js'
    import RFB from '../../lib/noVNC/core/rfb.js'
    import '../../lib/noVNC/core/input/keysym.js'
    export default {
        name: 'novnc',
        data () {
            return {
                rfb: null,
                url: 'ws://192.168.99.105:4444/vnc/2ffcd88435fd39b4f9089bf3763f3170',
                IsClean: false, //是否已断开并不可重新连接
                connectNum: 0 //重连次数
            }
        },
        methods: {
            // vnc连接断开的回调函数
            disconnectedFromServer (msg) {
                console.log('disconnectedFromServer')
                if(msg.detail.clean){
                    // 根据 断开信息的msg.detail.clean 来判断是否可以重新连接
                    this.contentVnc()
                } else {
                    //这里做不可重新连接的一些操作
                }
            },
            // 连接成功的回调函数
            connectedToServer() {
                console.log('success')
            },

            //连接vnc的函数
            connectVNC () {
                var
                    DEFAULT_HOST = '',
                    DEFAULT_PORT = '',
                    DEFAULT_PASSWORD = "",
                    DEFAULT_PATH = "websockify"
                ;
                var cRfb = null;
                var oTarget = document.getElementById("noVNC_canvas");
                let that = this
                if (!this.currentEquipment) {
                    return
                }
                let novncPort = this.currentEquipment.novncPort

                getNovncIp().then(function (resData) {
                    WebUtil.init_logging(WebUtil.getConfigVar("logging", "warn"));
                    var sHost = resData.data.content.ip || DEFAULT_HOST,
                        nPort = novncPort || DEFAULT_PORT,
                        sPassword = DEFAULT_PASSWORD,
                        sPath = DEFAULT_PATH
                    ;
                    cRfb = new RFB({
                        "target": oTarget,
                        //<span class="space" style="white-space:pre;display:inline-block;text-indent:2em;line-height:inherit;"> // 目标</span>
                        "focusContainer": top.document, // 鼠标焦点定位
                        "encrypt": WebUtil.getConfigVar("encrypt", window.location.protocol === "https:"),
                        "repeaterID": WebUtil.getConfigVar("repeaterID", ""),
                        "true_color": WebUtil.getConfigVar("true_color", true),
                        "local_cursor": WebUtil.getConfigVar("cursor", true),
                        "shared": WebUtil.getConfigVar("shared", true),
                        "view_only": WebUtil.getConfigVar("view_only", true),
                        "onFBUComplete": that._onCompleteHandler, // 回调函数
                        "onDisconnected": that._disconnected // 断开连接
                })
                    // console.log('sHost:' + sHost + '--nPort:' + nPort)
                    sHost = '192.168.99.105'
                    nPort = '4444'
                    sPassword = 'selenoid'
                    cRfb.connect(sHost, nPort, sPassword, sPath);
                })
            },
        },
        //在mounted周期里面连接vnc
        mounted () {
            //这时已经可以获取到dom元素
            console.log('在mounted周期里面连接vnc')
            this.connectVnc()
        }

    }
</script>