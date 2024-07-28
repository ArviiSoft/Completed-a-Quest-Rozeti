# 🗣️・AÇIKLAMA
**- Merhaba, bugün sizlere nasıl yeni gelen "Completed a Quest" rozetini oyun oynamadan alabileceğinizi anlatacağım. Bu yöntem tamamen güvenilir ve hiçbir riski yok bende şu an kullanıyorum.**
# 
#

# ❓・İŞLEYİŞ
1-) **Aşağıda verdiğim kod bloğunu kopyalayın.**

2-) **[Discord PTB](https://discord.com/api/downloads/distributions/app/installers/latest?channel=ptb&platform=win&arch=x64)'yi buradan indirin.**

3-) **PTB'yi açın, hesabınıza girin ve ayarlar simgesine basın. Ardından "Hediye Envanteri" kısmından Fortnite Quest'ini kabul edin.**

4-) **Bir tane ses kanalına girip not defteri ya da her hangi bir şeyi pencere olarak yayına verin ve yan hesabınızı sizle birlikte sese sokun.**

5-) **CTRL + SHIFT + I kombinasyonunu kullanarak PTB'nin ayarlar sekmesi içerisinde DevTools'u açın.**

6-) **DevTools'da Console kısmına gelin ve verilen kod bloğunu yapıştırıp enterlayın.**
# 
#

# 🧠・HATA ÇÖZÜMLERİ
**S:** "Warning: Don’t paste code into the DevTools Console that you don’t understand or haven’t reviewed yourself. This could allow attackers to steal your identity or take control of your computer. Please type ‘allow pasting’ below and hit Enter to allow pasting." hatası çıkıyor ne yapmalıyım?

**A:** Consolea ‘allow pasting’ yazıp enterlayın ardından kodu tekrar yapıştırıp enterlayın.


**S:** "TypeError: Cannot read properties of undefined (reading 'exports')
at <anonymous>:4:74" hatası alıyorum ne yapmalıyım?

**A:** Bu hata desteklenmeyen ve eski sürüm bir kod bloğu kullandığınızı gösterir. Verdiğim kod bloğu daha güncel.


**S:** "You don't have any uncompleted quests!" hatası alıyorum ne yapmalıyım?

**A:** Bu hata henüz bir Quest kabul etmediğinizi gösterir. Quest'i kabul edip tekrar deneyin.
#
#

# 🧑🏻‍💻・KOD BLOĞU
```js
let wpRequire;
window.webpackChunkdiscord_app.push([[ Math.random() ], {}, (req) => { wpRequire = req; }]);

let ApplicationStreamingStore, RunningGameStore, QuestsStore, ExperimentStore, FluxDispatcher, api
if(window.GLOBAL_ENV.SENTRY_TAGS.buildId === "366c746173a6ca0a801e9f4a4d7b6745e6de45d4") {
    ApplicationStreamingStore = Object.values(wpRequire.c).find(x => x?.exports?.default?.getStreamerActiveStreamMetadata).exports.default;
    RunningGameStore = Object.values(wpRequire.c).find(x => x?.exports?.default?.getRunningGames).exports.default;
    QuestsStore = Object.values(wpRequire.c).find(x => x?.exports?.default?.getQuest).exports.default;
    ExperimentStore = Object.values(wpRequire.c).find(x => x?.exports?.default?.getGuildExperiments).exports.default;
    FluxDispatcher = Object.values(wpRequire.c).find(x => x?.exports?.default?.flushWaitQueue).exports.default;
    api = Object.values(wpRequire.c).find(x => x?.exports?.getAPIBaseURL).exports.HTTP;
} else {
    ApplicationStreamingStore = Object.values(wpRequire.c).find(x => x?.exports?.Z?.getStreamerActiveStreamMetadata).exports.Z;
    RunningGameStore = Object.values(wpRequire.c).find(x => x?.exports?.ZP?.getRunningGames).exports.ZP;
    QuestsStore = Object.values(wpRequire.c).find(x => x?.exports?.Z?.getQuest).exports.Z;
    ExperimentStore = Object.values(wpRequire.c).find(x => x?.exports?.Z?.getGuildExperiments).exports.Z;
    FluxDispatcher = Object.values(wpRequire.c).find(x => x?.exports?.Z?.flushWaitQueue).exports.Z;
    api = Object.values(wpRequire.c).find(x => x?.exports?.tn?.get).exports.tn;
}

let quest = [...QuestsStore.quests.values()].filter(x => x.userStatus?.enrolledAt && !x.userStatus?.completedAt && new Date(x.config.expiresAt).getTime() > Date.now())[1]
let isApp = navigator.userAgent.includes("Electron/")
if(!isApp) {
    console.log("Tarayıcı artık DESTEKLENMİYOR! Lütfen masaüstü uygulamasını kullan. ( https://github.com/ArviiSoft/Completed-a-Quest-Rozeti )")
} else if(!quest) {
    console.log("Tamamlanmamış görevin yok. (Görevi kabul edip tekrar dene)")
} else {
    const pid = Math.floor(Math.random() * 30000) + 1000
    
    let applicationId, applicationName, secondsNeeded, secondsDone, canPlay
    if(quest.config.configVersion === 1) {
        applicationId = quest.config.applicationId
        applicationName = quest.config.applicationName
        secondsNeeded = quest.config.streamDurationRequirementMinutes * 60
        secondsDone = quest.userStatus?.streamProgressSeconds ?? 0
        canPlay = quest.config.variants.includes(2)
    } else if(quest.config.configVersion === 2) {
        applicationId = quest.config.application.id
        applicationName = quest.config.application.name
        canPlay = ExperimentStore.getUserExperimentBucket("2024-04_quest_playtime_task") > 0 && quest.config.taskConfig.tasks["PLAY_ON_DESKTOP"]
        const taskName = canPlay ? "PLAY_ON_DESKTOP" : "STREAM_ON_DESKTOP"
        secondsNeeded = quest.config.taskConfig.tasks[taskName].target
        secondsDone = quest.userStatus?.progress?.[taskName]?.value ?? 0
    }

    if(canPlay) {
        api.get({url: `/applications/public?application_ids=${applicationId}`}).then(res => {
            const appData = res.body[0]
            const exeName = appData.executables.find(x => x.os === "win32").name.replace(">","")
            
            const games = RunningGameStore.getRunningGames()
            const fakeGame = {
                cmdLine: `C:\\Program Files\\${appData.name}\\${exeName}`,
                exeName,
                exePath: `c:/program files/${appData.name.toLowerCase()}/${exeName}`,
                hidden: false,
                isLauncher: false,
                id: applicationId,
                name: appData.name,
                pid: pid,
                pidPath: [pid],
                processName: appData.name,
                start: Date.now(),
            }
            games.push(fakeGame)
            FluxDispatcher.dispatch({type: "RUNNING_GAMES_CHANGE", removed: [], added: [fakeGame], games: games})
            
            let fn = data => {
                let progress = quest.config.configVersion === 1 ? data.userStatus.streamProgressSeconds : Math.floor(data.userStatus.progress.PLAY_ON_DESKTOP.value)
                console.log(`Başarım İlerlemesi: ${progress}/${secondsNeeded}`)
                
                if(progress >= secondsNeeded) {
                    console.log("Başarım TAMAMLANDI! ( https://github.com/ArviiSoft/Completed-a-Quest-Rozeti )")
                    
                    const idx = games.indexOf(fakeGame)
                    if(idx > -1) {
                        games.splice(idx, 1)
                        FluxDispatcher.dispatch({type: "RUNNING_GAMES_CHANGE", removed: [fakeGame], added: [], games: []})
                    }
                    FluxDispatcher.unsubscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
                }
            }
            FluxDispatcher.subscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
            
            console.log(`Başarımın ( ${applicationName} ) adlı oyuna spamlanmaya başlandı. Lütfen ( ${Math.ceil((secondsNeeded - secondsDone) / 60)} ) dakika daha bekle.`)
        })
    } else {
        let realFunc = ApplicationStreamingStore.getStreamerActiveStreamMetadata
        ApplicationStreamingStore.getStreamerActiveStreamMetadata = () => ({
            id: applicationId,
            pid,
            sourceName: null
        })
        
        let fn = data => {
            let progress = quest.config.configVersion === 1 ? data.userStatus.streamProgressSeconds : Math.floor(data.userStatus.progress.STREAM_ON_DESKTOP.value)
            console.log(`Başarım İlerlemesi: ${progress}/${secondsNeeded}`)
            
            if(progress >= secondsNeeded) {
                console.log("Başarım TAMAMLANDI! ( https://github.com/ArviiSoft/Completed-a-Quest-Rozeti )")
                
                ApplicationStreamingStore.getStreamerActiveStreamMetadata = realFunc
                FluxDispatcher.unsubscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
            }
        }
        FluxDispatcher.subscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
        
        console.log(`Başarımın ( ${applicationName} ) adlı oyuna spamlanmaya başlandı. Lütfen ( ${Math.ceil((secondsNeeded - secondsDone) / 60)} ) dakika daha ses kanalında YAYIN YAP!`)
        console.log("NOT: Ses kanalına girip yayın açacağınız zaman en az farklı 1 hesap seste sizinle olsun. (Yan hesabınız da olabilir) ( https://github.com/ArviiSoft/Completed-a-Quest-Rozeti )")
    }
}
```
#
#

# 📞・İLETİŞİM
💙・**Discord:** arviis.

🔗・**Destek Sunucusu:** [Tıkla](https://discord.gg/3AfAFE5qYg)

💜・**Instagram:** [Tıkla](https://www.instagram.com/al.kann0/)
#
#

# 📷・GÖRSELLER
![1](https://github.com/user-attachments/assets/3722640d-d881-466d-9088-bca2e5c0c11d)

![2](https://github.com/user-attachments/assets/97a1e27a-f993-4aa5-b37b-4624ffdd8fe4)

![3](https://github.com/user-attachments/assets/9841da2f-f1a6-4e81-b063-95adfc4ac088)

# ❗・NOT
⁉️・Bu method hesabınızı KESİNLİKLE RİSKE SOKMAZ!
