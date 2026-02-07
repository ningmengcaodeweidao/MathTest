<template>
  <div id="app" class="container">
    <header>
      <h1>数学题生成工具</h1>
      <p class="subtitle">为一三班小朋友生成加减乘除练习题 | 支持打印、答案页和图片下载</p>
    </header>
    
    <div class="app">
      <!-- 控制面板 -->
      <div class="control-panel">
        <h2 class="panel-title">题目设置</h2>
        
        <div class="form-group">
          <label for="questionCount">题目数量：</label>
          <div class="range-slider-container">
            <input 
              type="range" 
              id="questionCountSlider" 
              v-model.number="questionCount" 
              min="5" 
              max="100" 
              step="5"
              class="slider"
            >
            <input 
              type="number" 
              id="questionCount" 
              v-model.number="questionCount" 
              min="5" 
              max="100" 
              class="number-input"
            >
            <span class="range-value">{{ questionCount }} 题</span>
          </div>
        </div>
        
        <div class="form-group">
          <label>包含的运算：</label>
          <div class="checkbox-group">
            <div class="checkbox-item">
              <input type="checkbox" id="addition" v-model="operations" value="+">
              <label for="addition">加法 (+)</label>
            </div>
            <div class="checkbox-item">
              <input type="checkbox" id="subtraction" v-model="operations" value="-">
              <label for="subtraction">减法 (-)</label>
            </div>
            <div class="checkbox-item">
              <input type="checkbox" id="multiplication" v-model="operations" value="×">
              <label for="multiplication">乘法 (×)</label>
            </div>
            <div class="checkbox-item">
              <input type="checkbox" id="division" v-model="operations" value="÷">
              <label for="division">除法 (÷)</label>
            </div>
          </div>
        </div>
        
        <div class="form-group">
          <label>数字范围：</label>
          <div class="range-slider-container">
            <div class="range-slider">
              <input 
                type="range" 
                v-model.number="minNumber" 
                :min="0" 
                :max="maxNumber - 1" 
                class="slider"
              >
              <input 
                type="range" 
                v-model.number="maxNumber" 
                :min="minNumber + 1" 
                :max="1000" 
                class="slider"
              >
            </div>
            <div class="range-inputs">
              <div class="range-input">
                <label for="minNumber">最小值</label>
                <input 
                  type="number" 
                  id="minNumber" 
                  v-model.number="minNumber" 
                  :min="0" 
                  :max="maxNumber - 1"
                  class="number-input"
                >
              </div>
              <div class="range-input">
                <label for="maxNumber">最大值</label>
                <input 
                  type="number" 
                  id="maxNumber" 
                  v-model.number="maxNumber" 
                  :min="minNumber + 1" 
                  :max="1000"
                  class="number-input"
                >
              </div>
            </div>
            <div class="range-display">
              范围: {{ minNumber }} - {{ maxNumber }}
            </div>
          </div>
        </div>
        
        <div class="form-group">
          <label>难度级别：</label>
          <div class="difficulty-options">
            <div 
              class="difficulty-option" 
              :class="{ active: difficulty === 'easy' }"
              @click="setDifficulty('easy')"
            >
              <div class="difficulty-dot easy"></div>
              <label>简单</label>
              <div class="difficulty-range">1-20</div>
            </div>
            <div 
              class="difficulty-option" 
              :class="{ active: difficulty === 'medium' }"
              @click="setDifficulty('medium')"
            >
              <div class="difficulty-dot medium"></div>
              <label>中等</label>
              <div class="difficulty-range">1-100</div>
            </div>
            <div 
              class="difficulty-option" 
              :class="{ active: difficulty === 'hard' }"
              @click="setDifficulty('hard')"
            >
              <div class="difficulty-dot hard"></div>
              <label>较难</label>
              <div class="difficulty-range">1-1000</div>
            </div>
          </div>
        </div>
        
        <div class="form-group">
          <label>布局设置：</label>
          <div class="layout-settings">
            <div class="layout-option">
              <div class="option-label">每行列数：</div>
              <div class="column-options">
                <div 
                  v-for="col in [2, 3, 4]" 
                  :key="col"
                  class="column-option" 
                  :class="{ active: columns == col }"
                  @click="columns = col"
                >
                  {{ col }}列
                </div>
              </div>
            </div>
            <div class="layout-option">
              <div class="option-label">显示序号：</div>
              <div class="toggle-switch" @click="showNumbers = !showNumbers">
                <div class="toggle-slider" :class="{ on: showNumbers }"></div>
                <span class="toggle-label">{{ showNumbers ? '开' : '关' }}</span>
              </div>
            </div>
            <div class="layout-option">
              <div class="option-label">打印答案：</div>
              <div class="toggle-switch" @click="printAnswers = !printAnswers">
                <div class="toggle-slider" :class="{ on: printAnswers }"></div>
                <span class="toggle-label">{{ printAnswers ? '开' : '关' }}</span>
              </div>
            </div>
          </div>
        </div>
        
        <div class="button-group">
          <button class="generate-btn" @click="generateQuestions">
            <span class="btn-icon">🎲</span>
            随机生成
          </button>
          <button class="print-btn" @click="printQuestions">
            <span class="btn-icon">🖨️</span>
            打印题目
          </button>
          <button class="image-btn" @click="downloadPrintImage" :disabled="isGeneratingImage">
            <span class="btn-icon" v-if="!isGeneratingImage">📷</span>
            <span class="btn-icon" v-else>⏳</span>
            {{ isGeneratingImage ? '生成中...' : '下载图片' }}
          </button>
          <button class="clear-btn" @click="clearQuestions">
            <span class="btn-icon">🗑️</span>
            清空题目
          </button>
        </div>
        
        <!-- 图片下载设置 -->
        <div v-if="questions.length > 0" class="form-group image-settings">
          <label>打印页面图片设置：</label>
          <div class="layout-settings">
            <div class="layout-option">
              <div class="option-label">图片质量：</div>
              <div class="quality-options">
                <div 
                  v-for="qual in qualityOptions" 
                  :key="qual.value"
                  class="quality-option" 
                  :class="{ active: imageQuality === qual.value }"
                  @click="imageQuality = qual.value"
                >
                  {{ qual.label }}
                </div>
              </div>
            </div>
            <div class="layout-option">
              <div class="option-label">包含答案：</div>
              <div class="toggle-switch" @click="includeAnswersInImage = !includeAnswersInImage">
                <div class="toggle-slider" :class="{ on: includeAnswersInImage }"></div>
                <span class="toggle-label">{{ includeAnswersInImage ? '是' : '否' }}</span>
              </div>
            </div>
            <div class="layout-option">
              <div class="option-label">图片格式：</div>
              <div class="image-format">
                <select v-model="imageFormat" class="format-select">
                  <option value="png">PNG (高质量)</option>
                  <option value="jpeg">JPEG (较小)</option>
                </select>
              </div>
            </div>
          </div>
        </div>
        
        <div class="stats">
          <div class="stat-item">
            <span class="stat-label">题目总数:</span>
            <span class="stat-value">{{ questions.length }}</span>
          </div>
          <div class="stat-item">
            <span class="stat-label">运算类型:</span>
            <span class="stat-value">{{ operations.length }} 种</span>
          </div>
          <div class="stat-item">
            <span class="stat-label">当前难度:</span>
            <span class="stat-value">{{ getDifficultyText(difficulty) }}</span>
          </div>
        </div>
      </div>
      
      <!-- 预览面板 -->
      <div class="preview-panel">
        <div class="preview-header">
          <h2 class="panel-title">题目预览</h2>
          <div class="preview-actions">
            <button 
              class="toggle-answers-btn" 
              @click="showAnswers = !showAnswers"
              :class="{ active: showAnswers }"
            >
              {{ showAnswers ? '👁️ 隐藏答案' : '👁️ 显示答案' }}
            </button>
            <button 
              class="toggle-numbers-btn" 
              @click="showNumbers = !showNumbers"
              :class="{ active: showNumbers }"
            >
              {{ showNumbers ? '🔢 隐藏序号' : '🔢 显示序号' }}
            </button>
          </div>
        </div>
        
        <div class="questions-container" :class="`cols-${columns}`">
          <div v-if="questions.length === 0" class="empty-state">
            <div class="empty-icon">📚</div>
            <h3>暂无数学题目</h3>
            <p>请设置题目参数并点击"随机生成"按钮</p>
            <div class="empty-hint">
              <p>💡 提示：可以调整题目数量、运算类型和数字范围</p>
            </div>
          </div>
          
          <div v-else>
            <div class="print-header print-only">
              <h2>数学练习题</h2>
              <div class="print-info">
                <div><strong>姓名：</strong>________________</div>
                <div><strong>班级：</strong>________________</div>
                <div><strong>日期：</strong>________________</div>
              </div>
              <p class="instructions"><strong>说明：</strong>请认真计算下列各题，将答案写在横线上。</p>
              <hr>
            </div>
            
            <div class="questions-grid">
              <div 
                v-for="(question, index) in questions" 
                :key="index" 
                class="question-item"
                :class="`difficulty-${question.difficulty}`"
              >
                <div v-if="showNumbers" class="question-number">
                  {{ index + 1 }}.
                </div>
                <div class="question-content">
                  <div class="question-text">
                    {{ question.expression }} = ________
                  </div>
                  <div v-if="!showAnswers" class="question-info">
                    <span class="op-badge">{{ question.operation }}</span>
                    <span class="diff-badge">{{ getDifficultyText(question.difficulty) }}</span>
                  </div>
                </div>
              </div>
            </div>
            
            <div class="print-footer print-only">
              <hr>
              <div class="print-summary">
                <div>完成时间: ________ 分钟</div>
                <div>得分: ________ / {{ questions.length }}</div>
              </div>
            </div>
          </div>
        </div>
        
        <!-- 答案区 -->
        <div class="answers-container" v-if="questions.length > 0 && showAnswers">
          <h3 class="answers-title">
            参考答案
            <span class="answers-count">(共 {{ questions.length }} 题)</span>
          </h3>
          <div class="answers-grid" :class="`cols-${columns}`">
            <div 
              v-for="(question, index) in questions" 
              :key="'answer-' + index" 
              class="answer-item"
            >
              <span v-if="showNumbers" class="answer-number">{{ index + 1 }}.</span>
              <span class="answer-text">
                {{ question.expression }} = 
                <span class="correct-answer">{{ question.answer }}</span>
              </span>
              <span class="answer-info">
                {{ question.operation }} | {{ getDifficultyText(question.difficulty) }}
              </span>
            </div>
          </div>
          
          <div class="answer-actions">
            <button @click="toggleAnswers" class="toggle-answers-btn">
              {{ showAnswers ? '👁️ 隐藏答案' : '👁️ 显示答案' }}
            </button>
            <p class="answer-hint">答案仅用于批改，打印时{{ printAnswers ? '会' : '不会' }}显示</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// 引入html2canvas库
import html2canvas from 'html2canvas';

export default {
  name: 'MathProblemGenerator',
  data() {
    return {
      questionCount: 20,
      // 默认只选中加减法
      operations: ['+', '-'],
      minNumber: 1,
      maxNumber: 100,
      difficulty: 'medium',
      columns: 3,
      showNumbers: true,
      showAnswers: true,
      printAnswers: false,
      questions: [],
      operationWeights: {
        '+': 1,
        '-': 1,
        '×': 1,
        '÷': 1
      },
      // 图片下载相关数据
      imageQuality: 2,
      includeAnswersInImage: false,
      imageFormat: 'png',
      qualityOptions: [
        { label: '标准', value: 1 },
        { label: '高清', value: 2 },
        { label: '超清', value: 3 }
      ],
      isGeneratingImage: false
    };
  },
  methods: {
    // 生成随机整数
    getRandomInt(min, max) {
      return Math.floor(Math.random() * (max - min + 1)) + min;
    },
    
    // 获取难度对应的文本
    getDifficultyText(difficulty) {
      const map = { easy: '简单', medium: '中等', hard: '较难' };
      return map[difficulty] || '中等';
    },
    
    // 设置难度级别
    setDifficulty(level) {
      this.difficulty = level;
      // 根据难度自动调整数字范围
      switch(level) {
        case 'easy':
          this.minNumber = 1;
          this.maxNumber = 20;
          break;
        case 'medium':
          this.minNumber = 1;
          this.maxNumber = 100;
          break;
        case 'hard':
          this.minNumber = 1;
          this.maxNumber = 1000;
          break;
      }
    },
    
    // 生成单个题目
    generateSingleQuestion() {
      // 确保至少选择了一个运算符
      if (this.operations.length === 0) {
        this.operations = ['+'];
      }
      
      // 根据选择的运算符随机选择一个
      const operation = this.operations[this.getRandomInt(0, this.operations.length - 1)];
      
      let num1, num2, answer, expression;
      
      // 根据难度设置不同的数字范围
      let minRange, maxRange;
      switch(this.difficulty) {
        case 'easy':
          minRange = 1;
          maxRange = Math.min(this.maxNumber, 20);
          break;
        case 'hard':
          minRange = Math.max(this.minNumber, 50);
          maxRange = Math.min(this.maxNumber, 1000);
          break;
        default: // medium
          minRange = this.minNumber;
          maxRange = Math.min(this.maxNumber, 100);
      }
      
      // 确保最小值不大于最大值
      const finalMin = Math.min(minRange, maxRange);
      const finalMax = Math.max(minRange, maxRange);
      
      // 根据运算符生成题目
      switch(operation) {
        case '+': {
          num1 = this.getRandomInt(finalMin, finalMax);
          num2 = this.getRandomInt(finalMin, finalMax);
          // 确保加法结果不超过范围上限的2倍
          const maxSum = Math.min(finalMax * 2, 1000);
          if (num1 + num2 > maxSum) {
            num2 = this.getRandomInt(finalMin, maxSum - num1);
          }
          answer = num1 + num2;
          expression = `${num1} + ${num2}`;
          break;
        }
          
        case '-': {
          num1 = this.getRandomInt(finalMin, finalMax);
          // 确保减法结果不为负数
          num2 = this.getRandomInt(finalMin, num1);
          answer = num1 - num2;
          expression = `${num1} - ${num2}`;
          break;
        }
          
        case '×': {
          // 乘法根据难度调整范围
          let multMax = this.difficulty === 'easy' ? 9 : 
                       this.difficulty === 'medium' ? 20 : 50;
          num1 = this.getRandomInt(1, Math.min(finalMax, multMax));
          num2 = this.getRandomInt(1, Math.min(finalMax, multMax));
          answer = num1 * num2;
          expression = `${num1} × ${num2}`;
          break;
        }
          
        case '÷': {
          // 除法确保能整除
          do {
            num2 = this.getRandomInt(1, this.difficulty === 'easy' ? 9 : 
                                   this.difficulty === 'medium' ? 20 : 50);
            answer = this.getRandomInt(1, Math.min(Math.floor(finalMax / num2), 20));
            num1 = num2 * answer;
          } while (num1 > finalMax || num1 < finalMin || num1 === 0);
          
          // 如果生成了无效的题目，重新生成一个加法题
          if (num1 === 0 || num2 === 0) {
            num1 = this.getRandomInt(finalMin, finalMax);
            num2 = this.getRandomInt(finalMin, finalMax);
            answer = num1 + num2;
            expression = `${num1} + ${num2}`;
            return {
              operation: '+',
              num1,
              num2,
              answer,
              expression,
              difficulty: this.difficulty
            };
          }
          
          expression = `${num1} ÷ ${num2}`;
          break;
        }
          
        default: {
          num1 = this.getRandomInt(finalMin, finalMax);
          num2 = this.getRandomInt(finalMin, finalMax);
          answer = num1 + num2;
          expression = `${num1} + ${num2}`;
        }
      }
      
      return {
        operation,
        num1,
        num2,
        answer,
        expression,
        difficulty: this.difficulty
      };
    },
    
    // 生成所有题目
    generateQuestions() {
      this.questions = [];
      
      // 确保至少有一个运算类型
      if (this.operations.length === 0) {
        this.operations = ['+'];
      }
      
      // 生成题目
      for (let i = 0; i < this.questionCount; i++) {
        this.questions.push(this.generateSingleQuestion());
      }
      
      // 显示答案
      this.showAnswers = true;
    },
    
    // 验证数字范围
    validateNumberRange() {
      // 确保最小值不大于最大值
      if (this.minNumber > this.maxNumber) {
        this.maxNumber = this.minNumber + 1;
      }
      
      // 确保范围合理
      this.minNumber = Math.max(0, Math.min(999, this.minNumber));
      this.maxNumber = Math.max(1, Math.min(1000, this.maxNumber));
    },
    
    // 验证题目数量
    validateQuestionCount() {
      if (this.questionCount < 5) {
        this.questionCount = 5;
      } else if (this.questionCount > 100) {
        this.questionCount = 100;
      }
    },
    
    // 生成打印页面的HTML
    generatePrintHTML(includeAnswers) {
      return `
        <!DOCTYPE html>
        <html>
        <head>
          <title>数学练习题</title>
          <meta charset="UTF-8">
          <style>
            * {
              margin: 0;
              padding: 0;
              box-sizing: border-box;
              font-family: 'Microsoft YaHei', Arial, sans-serif;
            }
            
            body { 
              padding: 20px; 
              line-height: 1.6;
              color: #000;
              background-color: white;
              width: 210mm; /* A4纸宽度 */
              min-height: 297mm; /* A4纸高度 */
              margin: 0 auto;
            }
            
            h1, h2 { 
              text-align: center; 
              color: #000;
              margin-top: 0;
              margin-bottom: 20px;
            }
            
            h1 {
              font-size: 28px;
            }
            
            h2 {
              font-size: 24px;
            }
            
            .header-info { 
              margin-bottom: 30px; 
            }
            
            .print-info {
              display: flex;
              justify-content: space-between;
              margin: 20px 0 30px;
              font-size: 18px;
              flex-wrap: wrap;
            }
            
            .print-info div {
              flex: 1;
              min-width: 150px;
              margin-bottom: 10px;
            }
            
            .instructions {
              margin: 20px 0;
              font-size: 18px;
              text-align: center;
            }
            
            .questions-grid {
              display: grid;
              grid-template-columns: repeat(${this.columns}, 1fr);
              gap: 20px;
              margin: 30px 0;
            }
            
            .question {
              font-size: 20px;
              padding: 15px 0;
              border-bottom: 1px dashed #ccc;
              display: flex;
              align-items: baseline;
              min-height: 50px;
              page-break-inside: avoid;
            }
            
            .question-number {
              margin-right: 8px;
              min-width: 30px;
              font-weight: bold;
            }
            
            .answers-section { 
              page-break-before: always; 
              margin-top: 40px; 
              padding-top: 20px;
              border-top: 2px solid #000;
            }
            
            .answers-grid {
              display: grid;
              grid-template-columns: repeat(${this.columns}, 1fr);
              gap: 15px;
              margin-top: 20px;
            }
            
            .answer { 
              font-size: 18px;
              padding: 10px 0;
              border-bottom: 1px solid #eee;
              display: flex;
              align-items: baseline;
              page-break-inside: avoid;
            }
            
            .answer-number {
              margin-right: 8px;
              min-width: 30px;
              font-weight: bold;
            }
            
            .correct-answer {
              font-weight: bold;
              color: #2ecc71;
            }
            
            hr { 
              border: none; 
              border-top: 2px solid #000; 
              margin: 20px 0; 
            }
            
            .print-summary {
              display: flex;
              justify-content: space-between;
              margin-top: 40px;
              font-size: 18px;
              padding-top: 20px;
              border-top: 1px solid #000;
            }
            
            .footer-note {
              text-align: center;
              margin-top: 30px;
              font-size: 16px;
              color: #666;
              font-style: italic;
            }
            
            @media print {
              .question { page-break-inside: avoid; }
              .answers-grid { page-break-inside: avoid; }
            }
            
            @page {
              margin: 1cm;
              size: A4;
            }
          </style>
        </head>
        <body>
          <h1>数学练习题</h1>
          <div class="header-info">
            <div class="print-info">
              <div><strong>姓名：</strong>________________</div>
              <div><strong>班级：</strong>________________</div>
              <div><strong>日期：</strong>________________</div>
            </div>
            <p class="instructions"><strong>说明：</strong>请认真计算下列各题，将答案写在横线上。</p>
            <hr>
          </div>
          
          <div class="questions-grid">
            ${this.questions.map((q, i) => `
              <div class="question">
                ${this.showNumbers ? `<span class="question-number">${i+1}.</span>` : ''}
                <span>${q.expression} = ________</span>
              </div>
            `).join('')}
          </div>
          
          <div class="print-summary">
            <div>完成时间: ________ 分钟</div>
            <div>得分: ________ / ${this.questions.length}</div>
          </div>
          
          ${includeAnswers ? `
            <div class="answers-section">
              <h2>参考答案</h2>
              <div class="answers-grid">
                ${this.questions.map((q, i) => `
                  <div class="answer">
                    ${this.showNumbers ? `<span class="answer-number">${i+1}.</span>` : ''}
                    <span>${q.expression} = <span class="correct-answer">${q.answer}</span></span>
                  </div>
                `).join('')}
              </div>
              <p class="footer-note">注：答案仅供参考，请仔细核对</p>
            </div>
          ` : ''}
        </body>
        </html>
      `;
    },
    
    // 打印题目
    printQuestions() {
      if (this.questions.length === 0) {
        alert('请先生成题目再打印');
        return;
      }
      
      // 生成打印内容 - 使用 printAnswers 控制是否包含答案
      const printContent = this.generatePrintHTML(this.printAnswers);
      
      // 打开新窗口并打印
      const printWindow = window.open('', '_blank');
      printWindow.document.write(printContent);
      printWindow.document.close();
      printWindow.focus();
      
      // 等待内容加载后打印
      setTimeout(() => {
        printWindow.print();
        // 打印后关闭窗口
        setTimeout(() => {
          printWindow.close();
        }, 100);
      }, 250);
    },
    
    // 下载打印页面为图片
    async downloadPrintImage() {
      if (this.questions.length === 0) {
        alert('请先生成题目再下载图片');
        return;
      }
      
      this.isGeneratingImage = true;
      
      try {
        // 生成打印内容 - 使用 includeAnswersInImage 控制是否包含答案
        const printContent = this.generatePrintHTML(this.includeAnswersInImage);
        
        // 创建临时iframe来渲染打印内容
        const iframe = document.createElement('iframe');
        iframe.style.cssText = `
          position: fixed;
          left: 0;
          top: 0;
          width: 210mm; /* A4纸宽度 */
          height: 297mm; /* A4纸高度 */
          border: none;
          visibility: hidden;
        `;
        
        document.body.appendChild(iframe);
        
        // 写入内容到iframe
        iframe.contentDocument.open();
        iframe.contentDocument.write(printContent);
        iframe.contentDocument.close();
        
        // 等待iframe内容加载完成
        await new Promise(resolve => {
          iframe.onload = resolve;
          // 如果iframe已经加载完成，立即解析
          if (iframe.contentDocument.readyState === 'complete') {
            resolve();
          }
        });
        
        // 获取iframe中的body元素
        const iframeBody = iframe.contentDocument.body;
        
        // 配置html2canvas选项
        const options = {
          scale: this.imageQuality,
          backgroundColor: '#ffffff',
          useCORS: true,
          allowTaint: true,
          foreignObjectRendering: false,
          logging: false,
          imageTimeout: 15000,
          width: iframeBody.scrollWidth,
          height: iframeBody.scrollHeight,
          windowWidth: iframeBody.scrollWidth,
          windowHeight: iframeBody.scrollHeight
        };
        
        // 生成截图
        const canvas = await html2canvas(iframeBody, options);
        
        // 移除临时iframe
        document.body.removeChild(iframe);
        
        // 生成文件名
        const date = new Date();
        const dateStr = `${date.getFullYear()}${(date.getMonth()+1).toString().padStart(2,'0')}${date.getDate().toString().padStart(2,'0')}`;
        const timeStr = `${date.getHours().toString().padStart(2,'0')}${date.getMinutes().toString().padStart(2,'0')}`;
        
        let fileName = `数学练习题_${this.questions.length}题_${dateStr}_${timeStr}`;
        if (this.includeAnswersInImage) {
          fileName += '_含答案';
        }
        
        // 根据选择的格式处理图片
        let mimeType, fileExt;
        switch(this.imageFormat) {
          case 'jpeg':
            mimeType = 'image/jpeg';
            fileExt = 'jpg';
            break;
          default:
            mimeType = 'image/png';
            fileExt = 'png';
        }
        
        // 转换并下载
        const imgData = canvas.toDataURL(mimeType, this.imageFormat === 'jpeg' ? 0.9 : 1.0);
        this.triggerDownload(imgData, `${fileName}.${fileExt}`);
        
        this.showToast('打印页面图片下载成功！');
        
      } catch (error) {
        console.error('生成打印页面图片失败:', error);
        alert('生成图片失败: ' + error.message);
      } finally {
        this.isGeneratingImage = false;
      }
    },
    
    // 触发文件下载
    triggerDownload(dataUrl, filename) {
      const link = document.createElement('a');
      link.href = dataUrl;
      link.download = filename;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },
    
    // 显示提示消息
    showToast(message) {
      // 移除已存在的提示
      const existingToast = document.querySelector('.toast-message');
      if (existingToast) {
        existingToast.remove();
      }
      
      // 创建新提示
      const toast = document.createElement('div');
      toast.className = 'toast-message';
      toast.textContent = message;
      toast.style.cssText = `
        position: fixed;
        top: 20px;
        right: 20px;
        background: #2ecc71;
        color: white;
        padding: 12px 20px;
        border-radius: 6px;
        box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        z-index: 1000;
        animation: slideIn 0.3s ease, fadeOut 0.3s ease 2.7s forwards;
      `;
      
      // 添加动画样式
      const style = document.createElement('style');
      style.textContent = `
        @keyframes slideIn {
          from { transform: translateX(100%); opacity: 0; }
          to { transform: translateX(0); opacity: 1; }
        }
        @keyframes fadeOut {
          from { opacity: 1; }
          to { opacity: 0; }
        }
      `;
      document.head.appendChild(style);
      
      document.body.appendChild(toast);
      
      // 3秒后自动移除
      setTimeout(() => {
        if (toast.parentNode) {
          toast.remove();
        }
        if (style.parentNode) {
          style.remove();
        }
      }, 3000);
    },
    
    // 清空题目
    clearQuestions() {
      if (confirm('确定要清空所有题目吗？')) {
        this.questions = [];
      }
    },
    
    // 切换答案显示
    toggleAnswers() {
      this.showAnswers = !this.showAnswers;
    }
  },
  mounted() {
    // 页面加载时生成一些示例题目
    this.generateQuestions();
  },
  watch: {
    // 验证数字范围
    minNumber() {
      this.validateNumberRange();
    },
    maxNumber() {
      this.validateNumberRange();
    },
    // 验证题目数量
    questionCount() {
      this.validateQuestionCount();
    },
    // 当难度变化时，自动调整数字范围
    difficulty(newVal) {
      switch(newVal) {
        case 'easy':
          this.minNumber = 1;
          this.maxNumber = 20;
          break;
        case 'medium':
          this.minNumber = 1;
          this.maxNumber = 100;
          break;
        case 'hard':
          this.minNumber = 1;
          this.maxNumber = 1000;
          break;
      }
    }
  }
};
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Microsoft YaHei', Arial, sans-serif;
}

body {
  background-color: #f5f7fa;
  color: #333;
  line-height: 1.6;
  padding: 20px;
}

.container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 20px;
}

header {
  text-align: center;
  margin-bottom: 30px;
  padding-bottom: 20px;
  border-bottom: 2px solid #4a6fa5;
}

h1 {
  color: #2c3e50;
  margin-bottom: 10px;
  font-size: 2.2rem;
}

.subtitle {
  color: #7f8c8d;
  font-size: 1.1rem;
  font-weight: 400;
}

.app {
  display: flex;
  flex-wrap: wrap;
  gap: 30px;
}

.control-panel {
  flex: 1;
  min-width: 320px;
  max-width: 400px;
  background-color: white;
  border-radius: 12px;
  padding: 25px;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.1);
  border: 1px solid #e0e0e0;
}

.preview-panel {
  flex: 2;
  min-width: 600px;
  background-color: white;
  border-radius: 12px;
  padding: 25px;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.1);
  border: 1px solid #e0e0e0;
}

.panel-title {
  font-size: 1.4rem;
  color: #3498db;
  margin-bottom: 20px;
  padding-bottom: 10px;
  border-bottom: 2px solid #eee;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.form-group {
  margin-bottom: 25px;
  padding-bottom: 20px;
  border-bottom: 1px solid #f0f0f0;
}

.form-group:last-child {
  border-bottom: none;
  margin-bottom: 0;
  padding-bottom: 0;
}

label {
  display: block;
  margin-bottom: 12px;
  font-weight: 600;
  color: #2c3e50;
  font-size: 1rem;
}

/* 滑块样式 */
.range-slider-container {
  margin-top: 10px;
}

.range-slider {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin-bottom: 15px;
}

.slider {
  width: 100%;
  height: 6px;
  border-radius: 3px;
  background: #ddd;
  outline: none;
  -webkit-appearance: none;
  appearance: none;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #3498db;
  cursor: pointer;
  border: 2px solid white;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

.slider::-moz-range-thumb {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #3498db;
  cursor: pointer;
  border: 2px solid white;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

.range-value {
  display: block;
  text-align: center;
  font-weight: 600;
  color: #3498db;
  margin-top: 8px;
  font-size: 1.1rem;
}

.range-inputs {
  display: flex;
  gap: 20px;
  margin-top: 15px;
}

.range-input {
  flex: 1;
}

.range-input label {
  font-size: 0.9rem;
  color: #666;
  margin-bottom: 8px;
  font-weight: 500;
}

.number-input {
  width: 100%;
  padding: 12px 15px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 1rem;
  transition: all 0.3s;
  text-align: center;
}

.number-input:focus {
  border-color: #3498db;
  outline: none;
  box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.2);
}

.range-display {
  text-align: center;
  margin-top: 10px;
  font-weight: 500;
  color: #7f8c8d;
  font-size: 0.95rem;
}

.checkbox-group {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
  margin-top: 8px;
}

.checkbox-item {
  display: flex;
  align-items: center;
  padding: 10px 12px;
  background-color: #f8f9fa;
  border-radius: 8px;
  border: 1px solid #e9ecef;
  transition: all 0.3s;
  cursor: pointer;
}

.checkbox-item:hover {
  background-color: #e9ecef;
  transform: translateY(-2px);
}

.checkbox-item input {
  margin-right: 10px;
  width: 18px;
  height: 18px;
  cursor: pointer;
}

.checkbox-item label {
  margin-bottom: 0;
  cursor: pointer;
  font-weight: 500;
  color: #495057;
}

.difficulty-options {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
  margin-top: 8px;
}

.difficulty-option {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 15px 10px;
  background-color: #f8f9fa;
  border-radius: 8px;
  border: 2px solid #e9ecef;
  cursor: pointer;
  transition: all 0.3s;
  text-align: center;
}

.difficulty-option:hover {
  background-color: #e9ecef;
  transform: translateY(-2px);
}

.difficulty-option.active {
  border-color: #3498db;
  background-color: #e3f2fd;
  box-shadow: 0 4px 8px rgba(52, 152, 219, 0.2);
}

.difficulty-dot {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  margin-bottom: 8px;
}

.difficulty-dot.easy {
  background-color: #2ecc71;
}

.difficulty-dot.medium {
  background-color: #f39c12;
}

.difficulty-dot.hard {
  background-color: #e74c3c;
}

.difficulty-range {
  font-size: 0.85rem;
  color: #7f8c8d;
  margin-top: 5px;
}

.layout-settings {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin-top: 10px;
}

.layout-option {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 0;
  border-bottom: 1px solid #f0f0f0;
}

.layout-option:last-child {
  border-bottom: none;
}

.option-label {
  font-weight: 500;
  color: #495057;
}

.column-options {
  display: flex;
  gap: 8px;
}

.column-option {
  padding: 8px 16px;
  background-color: #f8f9fa;
  border-radius: 6px;
  border: 1px solid #dee2e6;
  cursor: pointer;
  transition: all 0.3s;
  font-weight: 500;
  color: #495057;
}

.column-option:hover {
  background-color: #e9ecef;
}

.column-option.active {
  background-color: #3498db;
  color: white;
  border-color: #2980b9;
  box-shadow: 0 2px 4px rgba(52, 152, 219, 0.3);
}

.toggle-switch {
  display: flex;
  align-items: center;
  cursor: pointer;
  gap: 10px;
}

.toggle-slider {
  width: 50px;
  height: 26px;
  background-color: #ccc;
  border-radius: 13px;
  position: relative;
  transition: background-color 0.3s;
}

.toggle-slider:after {
  content: '';
  position: absolute;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  background-color: white;
  top: 2px;
  left: 2px;
  transition: transform 0.3s;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

.toggle-slider.on {
  background-color: #2ecc71;
}

.toggle-slider.on:after {
  transform: translateX(24px);
}

.toggle-label {
  font-weight: 500;
  color: #495057;
  min-width: 20px;
}

.button-group {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-top: 25px;
}

button {
  padding: 14px 20px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 600;
  transition: all 0.3s;
  flex: 1;
  min-width: 140px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.generate-btn {
  background-color: #3498db;
  color: white;
}

.generate-btn:hover {
  background-color: #2980b9;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(52, 152, 219, 0.3);
}

.print-btn {
  background-color: #2ecc71;
  color: white;
}

.print-btn:hover {
  background-color: #27ae60;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(46, 204, 113, 0.3);
}

.image-btn {
  background-color: #9b59b6;
  color: white;
}

.image-btn:hover {
  background-color: #8e44ad;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(155, 89, 182, 0.3);
}

.image-btn:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}

.clear-btn {
  background-color: #e74c3c;
  color: white;
}

.clear-btn:hover {
  background-color: #c0392b;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(231, 76, 60, 0.3);
}

.toggle-answers-btn, .toggle-numbers-btn {
  background-color: #f8f9fa;
  color: #495057;
  border: 1px solid #dee2e6;
  min-width: auto;
  padding: 10px 16px;
  font-weight: 500;
}

.toggle-answers-btn:hover, .toggle-numbers-btn:hover {
  background-color: #e9ecef;
  transform: translateY(-1px);
}

.toggle-answers-btn.active, .toggle-numbers-btn.active {
  background-color: #3498db;
  color: white;
  border-color: #2980b9;
}

.btn-icon {
  font-size: 1.2rem;
}

/* 图片设置样式 */
.image-settings {
  animation: fadeIn 0.5s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-10px); }
  to { opacity: 1; transform: translateY(0); }
}

.quality-options {
  display: flex;
  gap: 8px;
}

.quality-option {
  padding: 6px 12px;
  background-color: #f8f9fa;
  border-radius: 6px;
  border: 1px solid #dee2e6;
  cursor: pointer;
  transition: all 0.3s;
  font-size: 0.9rem;
  color: #495057;
  min-width: 50px;
  text-align: center;
}

.quality-option:hover {
  background-color: #e9ecef;
}

.quality-option.active {
  background-color: #9b59b6;
  color: white;
  border-color: #8e44ad;
  box-shadow: 0 2px 4px rgba(155, 89, 182, 0.3);
}

.format-select {
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 0.9rem;
  width: 100%;
  background-color: white;
  cursor: pointer;
  transition: all 0.3s;
}

.format-select:focus {
  border-color: #9b59b6;
  outline: none;
  box-shadow: 0 0 0 3px rgba(155, 89, 182, 0.2);
}

.stats {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
  margin-top: 25px;
  padding-top: 20px;
  border-top: 1px solid #eee;
}

.stat-item {
  text-align: center;
  padding: 12px;
  background-color: #f8f9fa;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.stat-label {
  font-size: 0.85rem;
  color: #7f8c8d;
}

.stat-value {
  font-weight: 600;
  color: #2c3e50;
  font-size: 1.1rem;
}

.preview-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 20px;
  flex-wrap: wrap;
  gap: 15px;
}

.preview-actions {
  display: flex;
  gap: 10px;
}

/* 题目容器样式 */
.questions-container {
  min-height: 500px;
  max-height: 600px;
  overflow-y: auto;
  padding: 20px;
  border: 1px solid #eee;
  border-radius: 10px;
  background-color: #f9f9f9;
  transition: all 0.3s;
}

/* 多列布局 */
.questions-container.cols-2 .questions-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
}

.questions-container.cols-3 .questions-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
}

.questions-container.cols-4 .questions-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 15px;
}

.questions-grid {
  transition: all 0.3s;
}

.question-item {
  padding: 18px 15px;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.05);
  display: flex;
  align-items: flex-start;
  transition: all 0.3s;
  border-left: 4px solid #3498db;
}

.question-item:hover {
  transform: translateY(-3px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}

.question-item.difficulty-easy {
  border-left-color: #2ecc71;
}

.question-item.difficulty-medium {
  border-left-color: #f39c12;
}

.question-item.difficulty-hard {
  border-left-color: #e74c3c;
}

.question-number {
  font-weight: bold;
  color: #3498db;
  margin-right: 10px;
  min-width: 30px;
  font-size: 1.1rem;
}

.question-content {
  flex: 1;
}

.question-text {
  font-size: 1.3rem;
  margin-bottom: 8px;
  font-weight: 600;
  color: #2c3e50;
  font-family: 'Courier New', monospace;
}

.question-info {
  display: flex;
  gap: 10px;
  font-size: 0.85rem;
  opacity: 0.8;
}

.op-badge, .diff-badge {
  padding: 3px 8px;
  border-radius: 4px;
  font-size: 0.8rem;
  font-weight: 500;
}

.op-badge {
  background-color: #e3f2fd;
  color: #1976d2;
}

.diff-badge {
  background-color: #f3e5f5;
  color: #7b1fa2;
}

/* 答案区样式 */
.answers-container {
  margin-top: 30px;
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 10px;
  border: 1px solid #e9ecef;
}

.answers-title {
  font-size: 1.3rem;
  color: #2c3e50;
  margin-bottom: 20px;
  display: flex;
  align-items: center;
  gap: 10px;
}

.answers-count {
  font-size: 1rem;
  color: #7f8c8d;
  font-weight: 400;
}

.answers-grid {
  margin: 15px 0;
}

.answers-grid.cols-2 {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
}

.answers-grid.cols-3 {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
}

.answers-grid.cols-4 {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
}

.answer-item {
  padding: 12px 15px;
  background-color: white;
  border-radius: 6px;
  display: flex;
  align-items: center;
  gap: 10px;
  border-left: 3px solid #2ecc71;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
}

.answer-number {
  font-weight: bold;
  color: #2ecc71;
  min-width: 25px;
}

.answer-text {
  font-size: 1rem;
  font-family: 'Courier New', monospace;
  flex: 1;
}

.correct-answer {
  font-weight: bold;
  color: #2ecc71;
  font-size: 1.1rem;
}

.answer-info {
  font-size: 0.8rem;
  color: #7f8c8d;
  opacity: 0.7;
}

.answer-actions {
  margin-top: 20px;
  text-align: center;
  padding-top: 20px;
  border-top: 1px solid #dee2e6;
}

.answer-hint {
  margin-top: 10px;
  font-size: 0.9rem;
  color: #7f8c8d;
}

/* 空状态样式 */
.empty-state {
  text-align: center;
  padding: 60px 20px;
  color: #7f8c8d;
}

.empty-icon {
  font-size: 4rem;
  margin-bottom: 20px;
  opacity: 0.5;
}

.empty-state h3 {
  margin-bottom: 10px;
  color: #95a5a6;
  font-size: 1.5rem;
}

.empty-state p {
  font-size: 1.1rem;
  margin-bottom: 20px;
}

.empty-hint {
  margin-top: 30px;
  padding: 15px;
  background-color: #e3f2fd;
  border-radius: 8px;
  max-width: 300px;
  margin-left: auto;
  margin-right: auto;
}

.empty-hint p {
  font-size: 0.95rem;
  margin: 0;
}

.print-only {
  display: none;
}

.print-header, .print-footer {
  text-align: center;
  margin-bottom: 20px;
}

.print-info {
  display: flex;
  justify-content: space-around;
  margin: 20px 0;
  flex-wrap: wrap;
}

.print-summary {
  display: flex;
  justify-content: space-between;
  margin-top: 20px;
  font-size: 1.1rem;
}

/* 响应式设计 */
@media (max-width: 1200px) {
  .container {
    max-width: 100%;
    padding: 10px;
  }
  
  .questions-container.cols-4 .questions-grid {
    grid-template-columns: repeat(3, 1fr);
  }
  
  .answers-grid.cols-4 {
    grid-template-columns: repeat(3, 1fr);
  }
  
  .quality-options {
    flex-wrap: wrap;
  }
  
  .quality-option {
    flex: 1;
    min-width: 0;
  }
}

@media (max-width: 992px) {
  .app {
    flex-direction: column;
  }
  
  .control-panel {
    min-width: 100%;
    max-width: 100%;
  }
  
  .preview-panel {
    min-width: 100%;
  }
  
  .questions-container.cols-4 .questions-grid,
  .questions-container.cols-3 .questions-grid {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .answers-grid.cols-4,
  .answers-grid.cols-3 {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .preview-header {
    flex-direction: column;
    align-items: flex-start;
  }
  
  .preview-actions {
    width: 100%;
    justify-content: center;
  }
}

@media (max-width: 768px) {
  .questions-container.cols-4 .questions-grid,
  .questions-container.cols-3 .questions-grid,
  .questions-container.cols-2 .questions-grid {
    grid-template-columns: 1fr;
  }
  
  .answers-grid.cols-4,
  .answers-grid.cols-3,
  .answers-grid.cols-2 {
    grid-template-columns: 1fr;
  }
  
  .checkbox-group {
    grid-template-columns: 1fr;
  }
  
  .difficulty-options {
    grid-template-columns: 1fr;
  }
  
  .range-inputs {
    flex-direction: column;
  }
  
  .button-group {
    flex-direction: column;
  }
  
  button {
    width: 100%;
  }
  
  .stats {
    grid-template-columns: 1fr;
  }
  
  h1 {
    font-size: 1.8rem;
  }
  
  .preview-header {
    flex-direction: column;
  }
  
  .preview-actions {
    flex-direction: column;
  }
  
  .image-format {
    width: 100%;
  }
  
  .format-select {
    width: 100%;
  }
}

/* 打印样式 */
@media print {
  .control-panel,
  .print-btn,
  .clear-btn,
  .generate-btn,
  .image-btn,
  .image-settings,
  header,
  .answers-container h3,
  .answer-actions,
  .toggle-answers-btn,
  .toggle-numbers-btn,
  .panel-title,
  .preview-actions,
  .question-info,
  .answer-info,
  .stats {
    display: none !important;
  }
  
  .print-only {
    display: block;
  }
  
  .preview-panel {
    box-shadow: none;
    padding: 0;
    min-width: 100%;
    width: 100%;
    border: none;
  }
  
  .questions-container {
    max-height: none;
    overflow: visible;
    border: none;
    padding: 0;
    min-height: auto;
    background-color: white;
  }
  
  .question-item {
    page-break-inside: avoid;
    border-left: none;
    box-shadow: none;
    padding: 15px 0;
    border-bottom: 1px dashed #ccc;
    margin-bottom: 0;
  }
  
  .question-text {
    font-size: 1.2rem;
  }
  
  body {
    background-color: white;
    padding: 10px;
  }
  
  .answers-container {
    page-break-before: always;
    margin-top: 40px;
    border-top: 2px solid #000;
    background-color: white;
    border: none;
    padding: 0;
  }
  
  .answers-container h2 {
    text-align: center;
    margin-bottom: 20px;
  }
  
  .answer-item {
    border: none;
    border-bottom: 1px solid #ddd;
    box-shadow: none;
    padding: 10px 0;
  }
  
  /* 打印时的列布局 */
  .questions-grid {
    display: grid !important;
  }
  
  .answers-grid {
    display: grid !important;
  }
}
</style>