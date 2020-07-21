<template>
    <div id="noVNC_all" class="vnc-card">
        <div id="noVNC_status_bar">
            <div id="noVNC_left_dummy_elem"></div>
            <div id="noVNC_status">Loading</div>
            <div id="noVNC_buttons">
                <input type=button value="Send CtrlAltDel"
                       id="sendCtrlAltDelButton" class="noVNC_shown">
                <span id="noVNC_power_buttons" class="noVNC_hidden">
        <input type=button value="Shutdown"
               id="machineShutdownButton">
        <input type=button value="Reboot"
               id="machineRebootButton">
        <input type=button value="Reset"
               id="machineResetButton">
      </span>
            </div>
        </div>
    </div>
</template>

<script>
    import * as WebUtil from '../../lib/noVNC/app/webutil.js';
    import RFB from '@novnc/novnc/core/rfb.js';
    export default {
        components:{
        },
        data() {
            return {
                rfb:null,
                desktopName:null
            };
        },
        methods: {
            connectVNC () {},
            updateDesktopName(e) {
                this.desktopName = e.detail.name;
            },
            credentials() {
                //var html;

                var form = document.createElement('form');
                form.innerHTML = '<label></label>';
                form.innerHTML += '<input type=password size=10 id="password_input">';
                form.onsubmit = this.setPassword;

                // bypass status() because it sets text content
                document.getElementById('noVNC_status_bar').setAttribute("class", "noVNC_status_warn");
                document.getElementById('noVNC_status').innerHTML = '';
                document.getElementById('noVNC_status').appendChild(form);
                document.getElementById('noVNC_status').querySelector('label').textContent = 'Password Required: ';
            },
            setPassword() {
                this.rfb.sendCredentials({ password: document.getElementById('password_input').value });
                return false;
            },
            sendCtrlAltDel() {
                this.rfb.sendCtrlAltDel();
                return false;
            },
            machineShutdown() {
                this.rfb.machineShutdown();
                return false;
            },
            machineReboot() {
                this.rfb.machineReboot();
                return false;
            },
            machineReset() {
                this.rfb.machineReset();
                return false;
            },
            status(text, level) {
                switch (level) {
                    case 'normal':
                    case 'warn':
                    case 'error':
                        break;
                    default:
                        level = "warn";
                }
                document.getElementById('noVNC_status_bar').className = "noVNC_status_" + level;
                document.getElementById('noVNC_status').textContent = text;
            },

            connected() {
                console.log("connected")
                document.getElementById('sendCtrlAltDelButton').disabled = false;
                if (WebUtil.getConfigVar('encrypt',
                    (window.location.protocol === "https:"))) {
                    this.status("Connected (encrypted) to " + this.desktopName, "normal");
                } else {
                    this.status("Connected (unencrypted) to " + this.desktopName, "normal");
                }
            },
            disconnected(e) {
                document.getElementById('sendCtrlAltDelButton').disabled = true;
                this.updatePowerButtons();
                if (e.detail.clean) {
                    this.status("Disconnected", "normal");
                } else {
                    this.status("Something went wrong, connection is closed", "error");
                }
            },
            updatePowerButtons() {
                var powerbuttons;
                powerbuttons = document.getElementById('noVNC_power_buttons');
                if (this.rfb.capabilities.power) {
                    powerbuttons.className= "noVNC_shown";
                } else {
                    powerbuttons.className = "noVNC_hidden";
                }
            }
        },
        mounted() {
            document.getElementById('sendCtrlAltDelButton').onclick = this.sendCtrlAltDel;
            document.getElementById('machineShutdownButton').onclick = this.machineShutdown;
            document.getElementById('machineRebootButton').onclick = this.machineReboot;
            document.getElementById('machineResetButton').onclick = this.machineReset;

            var url;
            url = 'ws://192.168.99.105:4444/vnc/07952908f40e9d7ef4dc5edaa4ae46de'
            console.log(url)
            const PASSWORD = 'selenoid';
            this.rfb = new RFB(document.querySelector('#noVNC_all'), url, {
                // 向vnc 传递的一些参数，比如说虚拟机的开机密码等
                credentials: {password: PASSWORD}
            });
            this.rfb.viewOnly = WebUtil.getConfigVar('view_only', false);
            this.rfb.addEventListener("connect",  this.connected);
            this.rfb.addEventListener("disconnect", this.disconnected);
            this.rfb.scaleViewport = true
            this.rfb.resizeSession = true
        }
    };
</script>
<style lang='scss' scoped>
    .vnc-card {
        height: 450px;
        width: 100%;
        display: flex;
        flex-direction: column;
        box-shadow: 0 1px 6px rgba(0, 0, 0, 0.12), 0 1px 4px rgba(0, 0, 0, 0.12);
    }
        #noVNC_status_bar {
        width: 100%;
        display:flex;
        justify-content: space-between;
    }

    #noVNC_status {
        color: #fff;
        font: bold 12px Helvetica;
        margin: auto;
    }

    .noVNC_status_normal {
        background: linear-gradient(#b2bdcd 0%,#899cb3 49%,#7e93af 51%,#6e84a3 100%);
    }

    .noVNC_status_error {
        background: linear-gradient(#c83737 0%,#899cb3 49%,#7e93af 51%,#6e84a3 100%);
    }

    .noVNC_status_warn {
        background: linear-gradient(#b4b41e 0%,#899cb3 49%,#7e93af 51%,#6e84a3 100%);
    }

    .noNVC_shown {
        display: inline;
    }
    .noVNC_hidden {
        display: none;
    }

    #noVNC_left_dummy_elem {
        flex: 1;
    }

    #noVNC_buttons {
        padding: 1px;
        flex: 1;
        display: flex;
        justify-content: flex-end;
    }
</style>