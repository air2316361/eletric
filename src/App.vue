<template>
  <div class="container">
    <h1 class="page-title">终末地电池二进制分流计算器</h1>
    <el-form :model="form" label-position="top">
      <el-form-item label="选择电池">
        <el-select v-model="form.recipe" style="width: 100%">
          <el-option
              v-for="item in recipes"
              :key="item.index"
              :label="item.label"
              :value="item.index"/>
        </el-select>
      </el-form-item>
      <el-form-item label="输入负载">
        <el-input-number
            v-model="form.load"
            :min="0"
            style="width: 100%"
            controls-position="right">
        </el-input-number>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" class="submit-btn" @click="handleSubmit">
          确认提交
        </el-button>
      </el-form-item>
    </el-form>
    <div class="description" v-if="descType !== 0">
      <p>{{ descType === 1 ? '您的基地不需要建造热能池' : '您的分流器排列顺序如下：' }}</p>
    </div>
    <div class="image-container" v-if="descType === 2">
      <el-row>
        <img v-for="item in totalBits" src="./assets/2.png" alt=""/>
      </el-row>
      <el-row>
        <template v-for="item in result">
          <img v-if="item === 1" src="./assets/1.png" alt=""/>
          <img v-else src="./assets/0.png" alt=""/>
        </template>
      </el-row>
      <el-row>
        <img v-for="item in totalBits" src="./assets/0.png" alt=""/>
      </el-row>
    </div>
    <div class="description" v-if="reservoirNum > 0">
      <p>此外，您还需要建造【{{ reservoirNum }}】个满载的热能池</p>
      <p>分流器输出功率：{{ output }}</p>
      <p>总输出功率：{{ totalOutput }}</p>
      <p>冗余：{{ redundancy }}</p>
    </div>
    <div class="description">请配合蓝图码【EF015i2OI880EuIeioe】食用</div>
  </div>
</template>

<script>

export default {
  data() {
    return {
      totalBits: 12,
      basePower: 200,
      recipes: [
        {
          index: 0,
          label: "低容谷地电池",
          power: 220,
          time: 40
        },
        {
          index: 1,
          label: "中容谷地电池",
          power: 420,
          time: 40
        },
        {
          index: 2,
          label: "高容谷地电池",
          power: 1100,
          time: 40
        },
        {
          index: 3,
          label: "低容武陵电池",
          power: 1600,
          time: 40
        }
      ],
      form: {
        recipe: 0,
        load: 1
      },
      descType: 0,
      reservoirNum: 0,
      result: [],
      output: 0.0,
      totalOutput: 0.0,
      redundancy: 0.0
    }
  },
  methods: {
    handleSubmit() {
      let load = this.form.load - this.basePower;
      this.reservoirNum = 0;
      if (load <= 0) {
        this.descType = 1;
        return;
      }
      this.descType = 2;
      const recipe = this.recipes[this.form.recipe]
      while (load > recipe.power) {
        ++this.reservoirNum;
        load -= recipe.power;
      }
      const maxPower = recipe.power * recipe.time / 2;
      const weights = [];
      for (let i = 0; i < this.totalBits; ++i) {
        if (i === 0) {
          weights[i] = maxPower / 2;
        } else {
          weights[i] = weights[i - 1] / 2
        }
      }
      let currentPower = 0;
      this.result =  new Array(this.totalBits).fill(0);
      for (let i = 0; i < this.totalBits; ++i) {
        const tempPower = currentPower + weights[i];
        if (tempPower <= load) {
          this.result[i] = 1;
          currentPower = tempPower;
        }
      }
      ++this.result[this.totalBits - 1];
      for (let i = this.totalBits - 1; i >= 0; --i) {
        if (this.result[i] < 2) {
          break
        }
        this.result[i] = 0;
        ++this.result[i - 1];
      }
      this.output = currentPower + weights[this.totalBits - 1];
      this.totalOutput = this.output + recipe.power * this.reservoirNum + this.basePower;
      this.redundancy = this.output - load;
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  background: #f5f7fa;
  min-height: 100vh;
  padding: 20px;
}

.container {
  max-width: 600px;
  margin: 0 auto;
  background: #fff;
  border-radius: 12px;
  padding: 24px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
}

.page-title {
  text-align: center;
  color: #303133;
  margin-bottom: 24px;
  font-size: 20px;
  font-weight: 600;
}

.image-container {
  width: 100%;
  margin-bottom: 24px;
  border-radius: 8px;
  overflow: hidden;
}

.image-container img {
  width: 8.3%;
  height: auto;
}

.description {
  color: #606266;
  line-height: 1.8;
  margin-bottom: 24px;
  font-size: 14px;
  text-align: justify;
}

.form-item {
  margin-bottom: 20px;
}

/* 移动端优化 */
@media screen and (max-width: 768px) {
  body {
    padding: 12px;
  }

  .container {
    padding: 16px;
    border-radius: 8px;
  }

  .page-title {
    font-size: 18px;
  }

  .description {
    font-size: 13px;
  }

  /* 让输入框在移动端更大，便于触摸 */
  .el-input__inner,
  .el-select .el-input__inner {
    height: 44px;
    font-size: 16px;
  }

  .el-input-number {
    width: 100%;
  }

  .el-button {
    width: 100%;
    height: 48px;
    font-size: 16px;
  }
}

.submit-btn {
  width: 100%;
  margin-top: 12px;
}
</style>
