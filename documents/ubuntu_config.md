#### ubuntu config
```
export JDK_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export MIUICAMERA_BUILDER_HOME=/home/lilongfei/work/miuicamera-builder
export ANDROID_HOME=/home/lilongfei/Android/Sdk
export PATH=$MIUICAMERA_BUILDER_HOME:$PATH
export PATH=$ANDROID_HOME/platform-tools:$PATH
#增加rooney定义的环境变量配置
export ROONEY_CONFIG=/home/lilongfei/.rooneyconfig/
export PATH=${ROONEY_CONFIG}:$PATH


#自定义命令别名
#miui camera builder
camerabuild() {
    if [[ -z "$1" ]]
        then echo "Product name must not be null or empty!"
	return
    fi
    if [[ -z "$2" ]]
        then echo "Label apk  watermark text empty!"
    fi
    miuicamera-builder --file Android.mk --api 28 --characteristic nosdcard -p "$1" --label "$2"
}

#cd相关
#alias work='cd ~/work'
#alias desktop='cd ~/desktop'
#alias downloads="cd ~/downloads"
#alias camera="cd ~/work/dev/cepheus/packages/apps/MiuiCamera"
#alias rooneyio="cd ~/work/roony_io/rooney.github.io"

#git相关
alias gpushdev='git push ssh://lilongfei@gerrit.pt.miui.com:29418/platform/packages/apps/MiuiCamera HEAD:refs/for/v9-phone-api2-dev'
alias gpushalpha='git push ssh://lilongfei@gerrit.pt.miui.com:29418/platform/packages/apps/MiuiCamera HEAD:refs/for/v9-phone-api2-alpha'
alias gpull="git pull --rebase"

#adb相关
alias camerapush="adb push build/apk/MiuiCamera.apk system/priv-app/MiuiCamera"
alias adbro="adb root"
alias adbre="adb remount"
alias adbdv="adb disable-verity"
alias adbrbb="adb reboot bootloader"
alias adbrb="adb reboot"
```