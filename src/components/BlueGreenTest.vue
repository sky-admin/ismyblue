<template>
  <div class="blue-green-test-wrapper">
    <div :style="containerStyle" class="blue-green-test-container">
      <div v-if="rounds < MAX_ROUNDS" class="blue-green-test-content">
        <transition name="fade-up" mode="out-in">
          <h1 v-if="showInitialMessage" key="initial" class="blue-green-test-title">
            <span class="background-white">测试你对蓝绿色的区分</span>
          </h1>
          <h1 v-else key="main" class="blue-green-test-title">
            <span class="background-white">你眼中的蓝色和大众眼中的蓝色有区别吗？</span>
          </h1>
        </transition>
      </div>
      <div v-else class="blue-green-test-content blue-green-test-result-screen">
        <Results
          :binPosition="binPosition"
          :count="count"
          :xCdf="xCdf"
          :yCdf="yCdf"
          :userThreshold="finalHue"
        />
      </div>
      <div v-if="rounds < MAX_ROUNDS" class="blue-green-test-button-container three-buttons">
        <button @click="selectColor('blue')" class="blue-green-test-button blue-button grow-button">
          这是蓝色
        </button>
        <button @click="reset" class="blue-green-test-button mid-reset-button grow-button">
          重新开始
        </button>
        <button
          @click="selectColor('green')"
          class="blue-green-test-button green-button grow-button"
        >
          这是绿色
        </button>
      </div>
      <div v-else class="blue-green-test-button-container two-buttons">
        <button
          @click="submitResults"
          class="blue-green-test-button submit-button grow-button"
          :disabled="submitted"
        >
          {{ submitted ? '已提交!' : '提交结果' }}
        </button>
        <button
          @click="showAbout = true"
          class="blue-green-test-button final-reset-button grow-button"
        >
          关于项目
        </button>
        <button @click="reset" class="blue-green-test-button final-reset-button grow-button">
          再来一次
        </button>
      </div>
    </div>
    <div v-if="submitted && showDemo" class="blue-green-test-submitted-message about-popup">
      <div class="about-content">
        <button @click="showDemo = false" class="close-button">&times;</button>
        <h2>谢谢您！在您离开之前...</h2>
        <p>
          如果方便的话，请告诉我们一些关于您的信息。我们将汇总不同人群对颜色分类的数据，制作综合分析图表。
        </p>
        <h3>您的母语</h3>
        <form @submit.prevent="submitDemographics">
          <div class="form-group">
            <label for="firstLanguage">不同语言对颜色的命名方式有所不同。您的母语是什么？</label>
            <div>
              <select id="firstLanguage" v-model="firstLanguage" class="form-control">
                <option value="Unspecified">请选择一种语言</option>
                <option value="Chinese">中文</option>
                <option value="English">英语</option>
                <option value="Spanish">西班牙语</option>
                <option value="Portuguese">葡萄牙语</option>
                <option value="French">法语</option>
                <option value="German">德语</option>
                <option value="Italian">意大利语</option>
                <option value="Greek">希腊语</option>
                <option value="Russian">俄语</option>
                <option value="Arabic">阿拉伯语</option>
                <option value="Japanese">日语</option>
                <option value="Korean">韩语</option>
                <option value="Thai">泰语</option>
                <option value="Vietnamese">越南语</option>
                <option value="Other with blue-green distinction">
                  其他有蓝色和绿色区分的语言
                </option>
                <option value="Other without blue-green distinction">
                  其他没有蓝绿色区分的语言
                </option>
              </select>
            </div>
          </div>
          <h3>您是色盲吗？</h3>
          <div class="form-group">
            <label for="colorBlindness">色盲可能会影响测试结果。</label>
            <div>
              <select id="colorBlindness" v-model="colorBlindness" class="form-control">
                <option value="dontknow">我不确定</option>
                <option value="no">否</option>
                <option value="red-green">是的，红绿色盲，具体类型未知</option>
                <option value="protanopia">是的，原色盲</option>
                <option value="protanomaly">是的，原色弱</option>
                <option value="deuteranopia">是的，绿色盲</option>
                <option value="deuteranomaly">是的，绿色弱</option>
                <option value="tritanopia">是的，蓝黄色盲</option>
                <option value="tritanomaly">是的，蓝黄色弱</option>
                <option value="achromatopsia">是的，全色盲</option>
                <option value="achromatomaly">是的，全色弱</option>
              </select>
            </div>
            <p><b>注意，这不是一个诊断工具。</b></p>
          </div>
          <button type="submit" class="submit-button-demo">提交</button>
        </form>
      </div>
    </div>
    <div v-if="showAbout" class="about-popup">
      <div class="about-content">
        <button @click="showAbout = false" class="close-button">&times;</button>
        <h2>关于这个网站</h2>
        <p>
          人们对自己看到的颜色有不同的命名。
          <a href="https://en.wikipedia.org/wiki/Sapir%E2%80%93Whorf_hypothesis" target="_blank"
          >语言可以影响我们如何记忆和命名颜色</a
          >。这是一个设计用于测量您个人蓝绿色边界的颜色命名测试。
        </p>
        <h2>测试的有效性</h2>
        <p>
          <b><i>本网站仅供娱乐用途。</i></b>
        </p>
        <p>
          颜色感知是很难测量的——视觉科学家使用专业校准设备来测量颜色感知。平面设计师使用物理色卡，如
          <a
            href="https://www.npr.org/2024/07/19/1197961103/pantone-colors-lawrence-herbert-stuart-semple-standards"
          >Pantone 生产的色卡</a
          >，以便能够明确地传达颜色。在这里，我们使用您的显示器或手机来测试您如何分类颜色，这远非完美，因为您的校准可能与我的不同。
        </p>
        <p>
          推断的有效性受到显示器校准、环境光和夜间模式等滤镜的限制。尽管存在这些限制，结果在<i>同一设备、同样环境光</i>下应具有良好的测试-重测可靠性，您可以通过多次测试来验证这一点。如果您想与朋友比较结果，请在同样的设备和环境光下进行。
        </p>
        <p>
          获得异常结果并不意味着您的视力有问题。这可能意味着您有一种独特的颜色命名方式，或者您的显示器和光线设置不同寻常。
        </p>
        <h2>技术细节</h2>
        <p>
          该测试要求您依次分类颜色。颜色通常以 HSL（色相、饱和度、亮度）色彩空间表示。色相 120 为绿色，色相 240 为蓝色。该测试重点关注 150 到 210 之间的蓝绿色色相。测试假设您对蓝色和绿色之间的响应以 S 型曲线表示。它会依次将 S 型曲线拟合到您的响应：
        </p>

        <img src="@/assets/formula.svg" alt="公式" />
        <br />
        <p>
          这相当于一个逻辑回归模型。测试使用最大后验估计（MAP）算法（具体而言，是用纯 JavaScript 实现的二阶牛顿法，不调用后台）将 S 型曲线拟合到您的响应，且对比例和偏移参数有一个模糊的先验。它使用拟合的曲线来确定接下来会呈现的颜色。它尝试聪明地选择采样新点的位置，重点放在您预测为中等信心的区域。为了提高结果的有效性，它会随机选择采样点，并使用噪声掩膜减轻视觉适应。
        </p>
        <p>
          这是一个曲线拟合，而不是二分查找。从理论上讲，如果您在中间色调中觉得自己在猜测，甚至猜错了也没关系。如果您在中间不一致，曲线拟合应该能够恢复，尽管您的估计阈值会有更大的误差。
        </p>

        <h2>结果</h2>
        <p>
          在早期实验中，我们发现人们的响应集中在 175 附近，巧合的是，这正好是 HTML 命名颜色“绿松石”的色相
          <span class="color-chip-turquoise mr-1"></span>。这很有趣，因为蓝色和绿色的名义边界在 180，即 HTML 命名颜色“青色”
          <span class="color-chip-cyan mr-1"></span>。这意味着大多数人的边界偏向认为青色是蓝色。
        </p>
        <h2>当我点击提交时会发生什么？</h2>
        <p>
          当您点击提交时，我们会匿名存储您的响应，以便日后汇总并测量总体命名曲线。我们不会存储任何能够识别您的个人信息。
        </p>
        <h2>这是谁做的？</h2>
        <p>
          原作者是 Patrick Mineault，一名神经科学和 AI 研究人员，使用 Claude 3.5 Sonnet 作为一个副项目制作了这个。原作者于 2014 年在麦吉尔大学获得视觉神经科学博士学位。
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import { createClient } from '@supabase/supabase-js'
import { MAX_ROUNDS, VERSION, BIN_POSITION, BIN_COUNT, X_CDF, Y_CDF } from '@/config'
import confetti from 'canvas-confetti'
import Results from './Results.vue'
import { fitSigmoid } from '@/utils/glmUtils'

import maskImage from '@/assets/mask.png'

const supabaseUrl = 'https://pnebergebsotpiobvvea.supabase.co'
const supabaseKey = process.env.SUPABASE_KEY

const supabase = createClient(supabaseUrl, supabaseKey)

export default {
  components: {
    Results
  },
  data() {
    return {
      MAX_ROUNDS: MAX_ROUNDS,
      currentHue: Math.random() > 0.5 ? 150 : 210,
      showInitialMessage: true,
      polarity: 0,
      rounds: 0,
      finalHue: 0,
      responses: [],
      userAgent: '',
      screenWidth: 0,
      screenHeight: 0,
      colorDepth: 0,
      pixelRatio: 1,
      timestamp: '',
      submitted: false,
      showMask: false,
      maskImageUrl: maskImage,
      binPosition: BIN_POSITION,
      count: BIN_COUNT,
      xCdf: X_CDF,
      yCdf: Y_CDF,
      showAbout: false,
      showDemo: false,
      anonymousId: this.generateAnonymousId(),
      firstLanguage: '',
      colorBlindness: false
    }
  },
  computed: {
    currentColor() {
      return `hsl(${this.currentHue}, 100%, 50%)`
    },
    bluerColor() {
      return `hsl(${this.finalHue + 5}, 100%, 50%)`
    },
    greenerColor() {
      return `hsl(${this.finalHue - 5}, 100%, 50%)`
    },
    containerStyle() {
      if (this.rounds === MAX_ROUNDS) {
        return {
          backgroundColor: 'white'
        }
      } else if (this.showMask) {
        return {
          backgroundColor: this.showMask ? 'transparent' : this.currentColor,
          backgroundImage: this.showMask ? `url(${this.maskImageUrl})` : 'none',
          backgroundRepeat: 'repeat',
          backgroundSize: 'auto'
        }
      } else {
        return {
          backgroundColor: this.currentColor
        }
      }
    }
  },
  methods: {
    selectColor(color) {
      this.responses.push({ hue: this.currentHue, response: color })

      // Get the new probe value
      const { b, newProbe, polarity } = fitSigmoid(
        this.responses.map((r) => r.hue),
        this.responses.map((r) => r.response === 'blue'),
        this.polarity,
        0.4
      )
      this.polarity = polarity === 1 ? -1 : 1
      this.currentHue = newProbe
      this.rounds++
      if (this.rounds === MAX_ROUNDS) {
        this.finalHue = 180 - b
        this.currentHue = this.finalHue
        confetti()
      }
      this.showMask = true
      setTimeout(() => {
        this.showMask = false
      }, 200)
    },
    reset() {
      this.anonymousId = this.generateAnonymousId()
      this.currentHue = Math.random() > 0.5 ? 150 : 210
      this.rounds = 0
      this.finalHue = 0
      this.showInitialMessage = true
      this.submitted = false
      this.responses = []
      this.showMask = false
      setTimeout(() => {
        this.showInitialMessage = false
      }, 2000)
    },
    generateAnonymousId() {
      return (
        Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15)
      )
    },
    async submitDemographics() {
      try {
        const { data, error } = await supabase.from('color_test_demo').insert([
          {
            anonymous_id: this.anonymousId,
            first_language: this.firstLanguage,
            color_blindness: this.colorBlindness
          }
        ])
        this.showDemo = false
      } catch (error) {
        console.error('Error submitting demographics:', error)
        alert('提交统计信息失败，请重试。')
      }
    },
    async submitResults() {
      this.gatherDeviceInfo()
      const now = new Date()
      this.timestamp = now.toISOString()

      // Create a local timestamp by adjusting for timezone offset
      const offsetMs = now.getTimezoneOffset() * 60 * 1000
      const localDate = new Date(now.getTime() - offsetMs)
      this.localTimestamp = localDate.toISOString()

      try {
        const payload = {
          anonymous_id: this.anonymousId,
          user_agent: this.userAgent,
          screen_width: this.screenWidth,
          screen_height: this.screenHeight,
          color_depth: this.colorDepth,
          pixel_ratio: this.pixelRatio,
          timestamp: this.timestamp,
          local_timestamp: this.localTimestamp,
          responses: this.responses,
          final_hue: this.finalHue,
          version: VERSION
        }
        const { data, error } = await supabase.from('color_test_results').insert([payload])
        console.log(payload)

        if (error) throw error

        this.submitted = true
        this.showDemo = true
      } catch (error) {
        console.error('Error submitting results:', error)
        alert('提交结果失败，请稍后重试')
      }
    },
    gatherDeviceInfo() {
      this.userAgent = navigator.userAgent
      this.screenWidth = window.screen.width
      this.screenHeight = window.screen.height
      this.colorDepth = window.screen.colorDepth
      this.pixelRatio = window.devicePixelRatio || 1
    }
  },
  mounted() {
    setTimeout(() => {
      this.showInitialMessage = false
    }, 2000)
  }
}
</script>

<style src="./BlueGreenTest.css" scoped />
<style scoped>
.color-chip-turquoise {
  display: inline-block;
  width: 1em;
  height: 1em;
  background-color: turquoise;
  border: 2px solid black;
  border-radius: 0.2em;
  margin-bottom: -0.2em;
}

.color-chip-cyan {
  display: inline-block;
  width: 1em;
  height: 1em;
  background-color: cyan;
  border: 2px solid black;
  border-radius: 0.2em;
  margin-bottom: -0.2em;
}

.about-popup {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.about-content {
  background-color: white;
  color: black;
  padding: 2rem;
  border-radius: 8px;
  max-width: 80%;
  max-height: 80%;
  overflow-y: auto;
  position: relative;
  font-family: 'Cabin', sans-serif;
  font-size: 0.9rem;
}

.close-button {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 1.5rem;
  background: none;
  border: none;
  cursor: pointer;
}

.about-content h2 {
  margin-top: 0;
  font-size: 1.2rem;
}

.about-content h3 {
  font-size: 1.2rem;
  margin-top: 1rem;
}

.about-content p {
  margin-bottom: 1rem;
}

.form-control {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-family: 'Cabin', sans-serif;
  font-size: 1rem;
  color: #333;
  background-color: #fff;
  appearance: none; /* Removes default styling in some browsers */
  -webkit-appearance: none; /* For older versions of Safari */
  -moz-appearance: none;
}

select.form-control {
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' fill='%23333' viewBox='0 0 16 16'%3E%3Cpath d='M7.247 11.14 2.451 5.658C1.885 5.013 2.345 4 3.204 4h9.592a1 1 0 0 1 .753 1.659l-4.796 5.48a1 1 0 0 1-1.506 0z'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 0.75rem center;
  background-size: 12px;
  padding-right: 2rem;
}

.form-control:focus {
  outline: none;
  border-color: #4a90e2;
  box-shadow: 0 0 0 2px rgba(74, 144, 226, 0.2);
}

.form-group {
  margin-bottom: 1.5rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
  color: #333;
}

/* Style specifically for the language dropdown */
#firstLanguage {
  border: 2px solid #4a90e2;
  transition: border-color 0.3s ease;
}

#firstLanguage:hover {
  border-color: #2a70c2;
}

/* Ensure text inputs match select styling */
input[type='text'].form-control {
  border: 2px solid #4a90e2;
}

/* Style for the submit button */
.submit-button-demo {
  background-color: #4a90e2;
  color: white;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
  transition: background-color 0.3s ease;
}

.submit-button-demo:hover {
  background-color: #2a70c2;
}
</style>
