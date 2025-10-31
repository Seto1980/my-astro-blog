---
title: "僕の資産および投資成績（2025年9月）"
meta_title: "資産ポートフォリオ"
description: "現在の金融資産ポートフォリオと、先月比の変動をインタラクティブなグラフで公開します。"
date: 2025-09-30T23:59:00+09:00
image: "/images/posts/08.png"
authors: ["ぐりっと"]
tags: ["投資", "資産公開", "家計簿"]
draft: false
---

こんにちは、ぐりっとです。
こちらは、僕の現在の金融資産ポートフォリオです。

`[資産構成]` と `[先月比変動]` のボタンを押すと、グラフが切り替わります。
データは2025年9月末時点のものです。毎月更新していく予定です！

<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/3.9.1/chart.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/chartjs-plugin-datalabels/2.2.0/chartjs-plugin-datalabels.min.js"></script>

<style>
  * {
    font-family: 'Noto Sans JP', sans-serif;
  }
  .chart-container {
    max-width: 800px;
    margin: 20px auto;
    background: white;
    border-radius: 12px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
    padding: 30px;
  }
  .controls { text-align: center; margin-bottom: 30px; }
  .btn-group { display: inline-flex; gap: 8px; background: #f1f3f4; padding: 4px; border-radius: 8px; }
  .toggle-btn { padding: 10px 20px; border: none; background: transparent; color: #666; border-radius: 6px; cursor: pointer; font-weight: 500; transition: all 0.3s ease; font-family: 'Noto Sans JP', sans-serif; }
  .toggle-btn.active { background: #05374b; color: white; box-shadow: 0 2px 4px rgba(5, 55, 75, 0.3); }
  .toggle-btn:hover:not(.active) { background: #e8eaed; }
  .chart-wrapper { position: relative; height: 500px; }
  .summary-info { margin-top: 20px; padding: 20px; background: #f8f9fa; border-radius: 8px; text-align: center; }
  .total-amount { font-size: 1.8em; font-weight: 700; color: #05374b; margin-bottom: 8px; }
  .month-change { font-size: 1.2em; font-weight: 500; }
  .positive { color: #2e8b57; }
  .negative { color: #dc3545; }
  @media (max-width: 768px) {
    .chart-container { padding: 20px; margin: 10px; }
    .chart-wrapper { height: 400px; }
    .btn-group { flex-direction: column; width: 100%; }
    .toggle-btn { width: 100%; margin: 2px 0; }
    .total-amount { font-size: 1.5em; }
  }
</style>

<div class="chart-container">
  <div class="controls">
    <div class="btn-group">
      <button class="toggle-btn active" data-chart="composition">資産構成</button>
      <button class="toggle-btn" data-chart="change">先月比変動</button>
    </div>
  </div>
  
  <div class="chart-wrapper">
    <canvas id="financialChart1"></canvas>
  </div>
  
  <div class="summary-info">
    <div class="total-amount">¥20,663,132</div>
    <div class="month-change negative">先月比 -¥129,495</div>
  </div>
</div>

<script is:inline>
  // Chart.js読み込みチェック
  if (typeof Chart === 'undefined') {
    console.error('Chart.js が読み込まれていません');
  }
  // データラベルプラグインの登録
  if (typeof ChartDataLabels !== 'undefined') {
    Chart.register(ChartDataLabels);
  }
  
  // 中央テキスト描画プラグイン
  const centerTextPlugin = {
    id: 'centerText',
    beforeDraw: function(chart) {
      const { ctx, width, height } = chart;
      ctx.restore();
      
      const fontSize = Math.min(width, height) / 12.5 * 0.9;
      ctx.font = `bold ${fontSize}px Noto Sans JP`;
      ctx.textBaseline = 'middle';
      ctx.textAlign = 'center';
      ctx.fillStyle = '#05374b';
      
      const line1 = '2025年9月';
      const line2 = '金融資産';
      const textX = width / 2;
      const textY = height / 2 - fontSize * 1.2;
      const lineHeight = fontSize * 1.2;
      
      ctx.fillText(line1, textX, textY - lineHeight / 2);
      ctx.fillText(line2, textX, textY + lineHeight / 2);
      ctx.save();
    }
  };
  
  // プラグインを登録
  Chart.register(centerTextPlugin);
  
  // データ定義
  const financialData1 = {
    composition: { "現金": 2179247, "投資信託": 9986647, "米国株": 3962662, "ETF": 3203993, "インドネシア株": 525880, "日本株": 382600, "マレーシア株": 209755, "仮想通貨": 212348, "企業型DC": 42095 },
    change: { "投資信託": 183863, "日本株": 37100, "ETF": 32729, "米国株": 5382, "現金": -354223, "インドネシア株": -19269, "仮想通貨": -10494, "マレーシア株": -4583, "企業型DC": 0 }
  };
  
  // カラーパレット
  const itemColors1 = {
    "現金": '#05374b', "投資信託": '#A2D7D4', "米国株": '#FFBADD', "ETF": '#BAB454', "インドネシア株": '#BADDFF', "日本株": '#C4C8E1', "マレーシア株": '#FFE4B5', "仮想通貨": '#E6E6FA', "企業型DC": '#D3D3D3'
  };
  
  let chartInstance1 = null;
  
  // チャート作成関数
  function createFinancialChart1(dataType) {
    const ctx = document.getElementById('financialChart1').getContext('2d');
    
    if (chartInstance1) {
      chartInstance1.destroy();
    }
    
    const data = financialData1[dataType];
    const labels = Object.keys(data);
    const values = Object.values(data);
    const backgroundColors = labels.map(label => itemColors1[label] || '#cccccc');
    
    const chartConfig = {
      type: 'doughnut',
      data: {
        labels: labels,
        datasets: [{
          data: values.map(Math.abs),
          backgroundColor: backgroundColors,
          borderWidth: 2,
          borderColor: '#ffffff'
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          datalabels: {
            color: '#ffffff',
            font: function(context) {
              const labelName = labels[context.dataIndex];
              if (dataType === 'change' && labelName === 'インドネシア株') {
                return { family: 'Noto Sans JP', size: 11, weight: 'bold' };
              }
              return { family: 'Noto Sans JP', size: 22, weight: 'bold' };
            },
            formatter: function(value, context) {
              const originalValue = values[context.dataIndex];
              const totalAbs = values.reduce((sum, v) => sum + Math.abs(v), 0);
              const percentage = ((Math.abs(originalValue) / totalAbs) * 100);
              const labelName = labels[context.dataIndex];
              
              if (percentage < 5) return '';
              
              if (dataType === 'composition') {
                return labelName + '\n(' + percentage.toFixed(1) + '%)';
              } else {
                const sign = originalValue >= 0 ? '+' : '';
                return labelName + '\n(' + sign + (originalValue / 10000).toFixed(0) + '万)';
              }
            },
            display: function(context) {
              const originalValue = values[context.dataIndex];
              const totalAbs = values.reduce((sum, v) => sum + Math.abs(v), 0);
              const percentage = ((Math.abs(originalValue) / totalAbs) * 100);
              return percentage >= 5;
            }
          },
          legend: {
            position: 'bottom',
            labels: {
              padding: 20,
              usePointStyle: true,
              font: { family: 'Noto Sans JP', size: 12 },
              generateLabels: function(chart) {
                const data = chart.data;
                if (data.labels.length && data.datasets.length) {
                  return data.labels.map((label, i) => {
                    const value = values[i];
                    const absValue = Math.abs(value);
                    const percentage = ((absValue / values.reduce((sum, v) => sum + Math.abs(v), 0)) * 100).toFixed(1);
                    
                    let displayText = label;
                    if (dataType === 'composition') {
                      displayText += ` (${percentage}%)`;
                    } else {
                      displayText += ` (${value >= 0 ? '+' : ''}${value.toLocaleString()}円)`;
                    }
                    
                    return {
                      text: displayText,
                      fillStyle: backgroundColors[i],
                      hidden: false,
                      index: i
                    };
                  });
                }
                return [];
              }
            }
          },
          tooltip: {
            backgroundColor: 'rgba(0,0,0,0.8)',
            titleFont: { family: 'Noto Sans JP', size: 14 },
            bodyFont: { family: 'Noto Sans JP', size: 12 },
            callbacks: {
              label: function(context) {
                const value = values[context.dataIndex];
                const absValue = Math.abs(value);
                if (dataType === 'composition') {
                  const percentage = ((absValue / values.reduce((sum, v) => sum + Math.abs(v), 0)) * 100).toFixed(1);
                  return `${context.label}: ¥${absValue.toLocaleString()} (${percentage}%)`;
                } else {
                  return `${context.label}: ${value >= 0 ? '+' : ''}¥${value.toLocaleString()}`;
                }
              }
            }
          }
        },
        animation: {
          animateRotate: true,
          duration: 1000
        },
        elements: {
          arc: {
            borderWidth: 2
          }
        }
      },
      plugins: [ChartDataLabels, centerTextPlugin]
    };
    chartInstance1 = new Chart(ctx, chartConfig);
  }
  
  // イベントリスナー
  document.addEventListener('DOMContentLoaded', function() {
    // DOMが読み込まれた後にチャートを作成
    createFinancialChart1('composition');
    
    // ボタンイベント
    document.querySelectorAll('.toggle-btn').forEach(btn => {
      btn.addEventListener('click', function() {
        document.querySelectorAll('.toggle-btn').forEach(b => b.classList.remove('active'));
        this.classList.add('active');
        const chartType = this.getAttribute('data-chart');
        createFinancialChart1(chartType);
      });
    });
  });
</script>