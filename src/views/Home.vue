<template>
  <v-container>
    <v-row justify="center">
      <v-col lg="10" xl="8">
        <v-card>
          <v-row no-gutters>
            <v-col cols="auto">
              <v-card-title class="text-h4">{{ today }}</v-card-title>
              <v-divider/>
              <v-card-subtitle>今日天气<v-btn icon :loading="ld0" @click="getWeather"><v-icon>mdi-refresh</v-icon></v-btn></v-card-subtitle>
              <v-card-text>{{ weather }}</v-card-text>
            </v-col>
            <v-divider vertical/>
            <v-col>
              <v-card-subtitle>习语江句<v-btn icon :loading="ld4" @click="golden"><v-icon>mdi-refresh</v-icon></v-btn></v-card-subtitle>
              <v-card-text>
                <p class="gold">{{ goldText }}</p>
                <p class="text-right ma-0">{{ goldFrom }}</p>
              </v-card-text>
              <v-divider/>
              <v-card-subtitle>高考倒计时</v-card-subtitle>
              <v-card-text>
                {{ NaN }}
              </v-card-text>
            </v-col>
          </v-row>
        </v-card>
      </v-col>
    </v-row>
    <v-row justify="center">
      <v-col lg="10" xl="8">
        <v-card>
          <v-card-title>
            快速操作
          </v-card-title>
          <v-card-text>
            <v-row justify="space-around">
              <v-col cols="3">
                <v-btn block color="primary" outlined @click="openFtp" :loading="ld1">打开FTP</v-btn>
              </v-col>
              <v-col cols="3">
                <v-btn block color="primary" outlined @click="openCourses" :loading="ld2">打开课件目录</v-btn>
              </v-col>
              <v-col cols="3">
                <v-btn block color="primary" outlined @click="closeAll" :loading="ld3">关闭所有窗口</v-btn>
              </v-col>
            </v-row>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import { Vue, Component } from 'vue-property-decorator'
import { shell, remote } from 'electron'
import { promisify } from 'util'
import { exec } from 'child_process'
import path from 'path'
import fs from 'fs-extra'

/* global __static */

const execAsync = promisify(exec)

@Component
export default class Home extends Vue {
  today!: string
  weather = '获取天气数据中'
  ld0 = false
  ld1 = false
  ld2 = false
  ld3 = false
  ld4 = false
  golds: string[][] = []
  goldText = '续命中'
  goldFrom = '扬州'

  created () {
    const now = new Date()
    this.today = `${now.getMonth() + 1}月${now.getDate()}日 星期${'日一二三四五六'[now.getDay()]}`
    this.golden()
    this.getWeather()
    const i = setInterval(() => {
      this.golden()
    }, 10000)
    this.$on('hooks:beforeDestory', () => clearInterval(i))
  }

  open (url: string) {
    shell.openExternal(url)
  }

  getWeather () {
    this.ld0 = true
    fetch('https://api.openweathermap.org/data/2.5/onecall?lat=29.95&lon=121.72&appid=2aa1712947719c02292a5bd3d2747b26&lang=zh_cn')
      .then(resp => resp.json())
      .then(data => {
        this.weather = data.daily[0].weather[0].description
      })
      .catch(e => {
        console.log(e)
        this.weather = '获取天气数据失败'
      })
      .finally(() => {
        this.ld0 = false
      })
  }

  async openFtp () {
    try {
      this.ld1 = true
      await execAsync('EXPLORER.EXE /root,ftp://ftp.zhzx.net.cn/%BD%CC%CA%A6%D7%CA%D4%B4/')
    } catch (e) {
      console.log(e)
    } finally {
      this.ld1 = false
    }
  }

  async openCourses () {
    try {
      this.ld2 = true
      const desktop = remote.app.getPath('desktop')
      await execAsync(`EXPLORER.EXE /root,${desktop}\\课件\\`)
    } catch (e) {
      console.log(e)
    } finally {
      this.ld2 = false
    }
  }

  async closeAll () {
    try {
      this.ld3 = true
      await execAsync('TASKKILL /F /IM POWERPNT.EXE').catch(() => 0)
      await execAsync('TASKKILL /F /IM WINWORD.EXE').catch(() => 0)
      await execAsync('TASKKILL /F /IM EXCEL.EXE').catch(() => 0)
      await execAsync('TASKKILL /F /IM EasiNote.exe').catch(() => 0)
    } catch (e) {
      console.log(e)
    } finally {
      this.ld3 = false
    }
  }

  async golden () {
    this.ld4 = true
    try {
      if (!this.golds.length) {
        const file = path.join(__static, 'mottos.txt')
        const content = await fs.readFile(file)
        this.golds = content
          .toString()
          .split('\n')
          .map(x => x.trim())
          .join('\n')
          .split('\n\n')
          .filter(x => x.length)
          .map(x => {
            const y = x.split('\n')
            const z = y.pop() || '出处：维基语录'
            return [y.join('\n'), z]
          })
      }
      const idx = Math.floor(Math.random() * this.golds.length)
      const t = this.golds.splice(idx, 1)
      this.goldText = t[0][0]
      this.goldFrom = t[0][1]
    } catch (e) {
      this.goldText = '你们啊，不要想……喜欢……这…欸弄个大新闻，说现在已经钦定了，再把我批判一番。'
      this.goldFrom = '2000年受有线新闻台记者张宝华采访'
    } finally {
      this.ld4 = false
    }
  }
}
</script>

<style scoped>
.gold {
  color: red;
  font-family: 'KaiTi', Times, serif;
  font-size: 1.5rem;
}
</style>
