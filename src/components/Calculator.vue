<template>
  <div>
    <p>推定市場価格を入力: {{ formattedTotalSilver }}</p>
    <input v-model="totalSilver" @input="restrictToNumbers" placeholder="edit me" />

    <p>全体の報酬: {{ totalReward }}</p>
    <p>参加者</p>
    <div v-for="(item, index) in items" :key="index" class="item">
      <input v-model="item.name" placeholder="名前" />
      <select v-model="item.percentage" tabindex="-1">
        <option v-for="percentageOption in percentageOptions" :value="percentageOption" :key="percentageOption">
          {{ percentageOption }}%
        </option>
      </select>
      {{ calculateReward(item) }}
    </div>

    <button @click="addItem">＋</button>

    <p>結果</p>
    <div>
      <textarea v-model="resultText" style="width:236px; height:236px;"></textarea>
    </div>

  </div>
</template>

<script>

export default {
  name: 'Calculator-Gank',
  props: {
    msg: String
  },
  data() {
    return {
      totalSilver: '',
      items: [
        { name: '', percentage: 0, reward: ''},
        { name: '', percentage: 0, reward: ''},
        { name: '', percentage: 0, reward: ''},
        { name: '', percentage: 0, reward: ''},
        { name: '', percentage: 0, reward: ''},
      ],
      percentageOptions: [0, 10, 20, 30, 40, 50, 60, 70, 80, 90, 100],
      resultText: '',
    }
  },
  computed: {
    formattedTotalSilver() {
      // 3桁ごとにカンマで区切る
      return this.totalSilver.replace(/\B(?=(\d{3})+(?!\d))/g, ',');
    },
    totalReward() {
      if (this.totalSilver === '') {
        return '0';
      } else {
        // totalSilverの0.8倍の値を計算する
        const reward = Math.floor(this.totalSilver * 0.8);
        return reward.toLocaleString();
      }
    },
    totalRewardWithNonZeroPercentage() {
      return this.items.reduce((rewardTotal, item) => {
        if (item.percentage !== 0 && !isNaN(parseFloat(item.reward))) {
          return rewardTotal + parseFloat(item.reward);
        } else {
          return rewardTotal;
        }
      }, 0);
    },
  },
  methods: {
    restrictToNumbers(event) {
      // 数値以外の文字を削除する
      this.totalSilver = event.target.value.replace(/\D/g, '');
    },
    addItem() {
      this.items.push({ name: '', percentage: 0, reward: '' });
    },
    calculateReward(item) {
      const nonEmptyNameItemsCount = this.items.filter(i => i.name !== '').length;
      const nonPercentageItemsCount = this.items.filter(i => i.name !== '' && i.percentage === 0).length;
      if (item.name !== '' && nonEmptyNameItemsCount !== 0) {
        const Reward = Math.floor(this.totalReward.replace(/,/g, '')) / nonEmptyNameItemsCount;
        if (item.percentage !== 0) {
          const individualReward = Reward + Reward * (item.percentage / 100);
          item.reward = individualReward
          this.resultText = this.items
            .filter(item => item.name !== '')
            .map(item => `${item.name} : ${item.reward} (+${item.percentage}%)`)
            .join('\n');
          return Math.floor(individualReward).toLocaleString();
        } else {
          const individualReward = (Math.floor(this.totalReward.replace(/,/g, '')) - this.totalRewardWithNonZeroPercentage) / nonPercentageItemsCount;
          item.reward = individualReward
          this.resultText = this.items
            .filter(item => item.name !== '')
            .map(item => `${item.name} : ${item.reward} (+${item.percentage}%)`)
            .join('\n');
          return Math.floor(individualReward).toLocaleString();
        }
      } else {
        return '';
      }
    },
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
