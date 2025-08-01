<template>
  <div class="monitor-container">
    <div class="header">
      <h1 class="title">监控面板</h1>
      <div class="stats">
        <span class="stat-item">总请求数: {{ monitorLogs.length }}</span>
      </div>
    </div>
    
    <div class="table-container">
      <table class="monitor-table">
        <thead>
          <tr>
            <th>UUID</th>
            <th>通道</th>
            <th>状态</th>
            <th>参数</th>
            <th>耗时(ms)</th>
            <th>结果</th>
            <th>开始时间</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(log, index) in monitorLogs" :key="index" :class="getRowClass(log)">
            <td class="uuid-cell">{{ formatUuid(log.id) }}</td>
            <td class="channel-cell">{{ log.channel }}</td>
            <td class="status-cell">
              <span :class="getStatusClass(log.status)">{{ getStatusText(log.status) }}</span>
            </td>
            <td class="args-cell" :title="JSON.stringify(log.args)">{{ formatArgs(log.args) }}</td>
            <td class="timestamp-cell">{{ formatTimestamp(log) }}</td>
            <td class="result-cell" :title="JSON.stringify(log.result)">{{ formatResult(log.result) }}</td>
            <td class="time-cell">{{ formatTime(log.startTime) }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>
<script lang="ts" setup>
import { onMounted, ref } from 'vue';

const monitorLogs = ref<any[]>([]);

// 格式化函数
const formatUuid = (uuid: string) => {
  return uuid ? uuid.substring(0, 8) + '...' : '-';
};

const formatArgs = (args: any) => {
  if (!args) return '-';
  const str = JSON.stringify(args);
  return str.length > 50 ? str.substring(0, 50) + '...' : str;
};

const formatResult = (result: any) => {
  if (!result) return '-';
  const str = JSON.stringify(result);
  return str.length > 50 ? str.substring(0, 50) + '...' : str;
};

const formatTime = (timestamp: number) => {
  if (!timestamp) return '-';
  return new Date(timestamp).toLocaleTimeString();
};

const formatTimestamp = (log: any) => {
  if(log.status === 'pending'){
    return 'pending'
  }
  return (log.timestamp / 1000).toFixed(2);
};

const getStatusText = (status: string) => {
  const statusMap: Record<string, string> = {
    'pending': '进行中',
    'fullfilled': '已完成',
    'rejected': '已失败'
  };
  return statusMap[status] || status;
};

const getStatusClass = (status: string) => {
  return `status-${status}`;
};

const getRowClass = (log: any) => {
  return `row-${log.status}`;
};

onMounted(() => {
  require('electron').ipcRenderer.on('monitor:data', (_, data) => {
    
      if (data.status === 'pending') {
        data.startTime = performance.now() // 记录实际开始时间
        monitorLogs.value.push(data)
      }
      if (data.status === 'fullfilled') {
        const index = monitorLogs.value.findIndex(item => item.id === data.id)
        const pendingData = { ...monitorLogs.value[index] }
        if (index !== -1) {
          data.startTime = pendingData.startTime
          console.log({...pendingData})
          data.timestamp = data.timestamp - pendingData.timestamp // 计算耗时
          monitorLogs.value[index] = data
        }
      }
      if (data.status === 'rejected') {
        const index = monitorLogs.value.findIndex(item => item.id === data.id)
        if (index !== -1) {
          const pendingData = { ...monitorLogs.value[index] }
          data.startTime = pendingData.startTime
          data.timestamp = data.timestamp - pendingData.timestamp
          monitorLogs.value[index] = data
        }
      }
      
      console.log(monitorLogs.value)
    })
});
</script>

<style scoped>
.monitor-container {
  padding: 20px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
  padding: 20px;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border-radius: 15px;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.title {
  color: white;
  font-size: 2.5rem;
  font-weight: 600;
  margin: 0;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

.stats {
  display: flex;
  gap: 20px;
}

.stat-item {
  color: white;
  font-size: 1.1rem;
  padding: 8px 16px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 20px;
  backdrop-filter: blur(5px);
}

.table-container {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 15px;
  overflow: hidden;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
}

.monitor-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 14px;
}

.monitor-table thead {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.monitor-table th {
  padding: 16px 12px;
  color: white;
  font-weight: 600;
  text-align: center;
  font-size: 13px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  border: none;
}

.monitor-table tbody tr {
  transition: all 0.3s ease;
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);
}

.monitor-table tbody tr:hover {
  background-color: rgba(102, 126, 234, 0.1);
  transform: translateY(-1px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.monitor-table td {
  padding: 12px;
  border: none;
  vertical-align: middle;
  text-align: center;
  max-width: 200px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* 特定列样式 */
.uuid-cell {
  font-family: 'Courier New', monospace;
  color: #666;
  font-size: 12px;
}

.channel-cell {
  font-weight: 600;
  color: #333;
}

.status-pending {
  background: linear-gradient(135deg, #ffeaa7, #fdcb6e);
  color: #e17055;
  padding: 4px 12px;
  border-radius: 15px;
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.status-fullfilled {
  background: linear-gradient(135deg, #00b894, #00cec9);
  color: white;
  padding: 4px 12px;
  border-radius: 15px;
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.status-rejected {
  background: linear-gradient(135deg, #e17055, #d63031);
  color: white;
  padding: 4px 12px;
  border-radius: 15px;
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.args-cell, .result-cell {
  font-family: 'Courier New', monospace;
  font-size: 12px;
  color: #555;
  cursor: help;
}

.timestamp-cell {
  font-weight: 600;
  color: #e17055;
}

.time-cell {
  color: #666;
  font-size: 12px;
}

/* 行状态样式 */
.row-pending {
  background: rgba(255, 234, 167, 0.3);
  border-left: 4px solid #fdcb6e;
}

.row-fullfilled {
  background: rgba(0, 184, 148, 0.1);
  border-left: 4px solid #00b894;
}

.row-rejected {
  background: rgba(225, 112, 85, 0.1);
  border-left: 4px solid #e17055;
}

/* 响应式设计 */
@media (max-width: 1200px) {
  .monitor-table {
    font-size: 12px;
  }
  
  .monitor-table th,
  .monitor-table td {
    padding: 8px 6px;
  }
  
  .title {
    font-size: 2rem;
  }
}

@media (max-width: 768px) {
  .monitor-container {
    padding: 10px;
  }
  
  .header {
    flex-direction: column;
    gap: 15px;
    text-align: center;
  }
  
  .monitor-table {
    font-size: 11px;
  }
  
  .monitor-table th,
  .monitor-table td {
    padding: 6px 4px;
  }
}

/* 滚动条美化 */
.table-container::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

.table-container::-webkit-scrollbar-track {
  background: rgba(0, 0, 0, 0.1);
  border-radius: 4px;
}

.table-container::-webkit-scrollbar-thumb {
  background: linear-gradient(135deg, #667eea, #764ba2);
  border-radius: 4px;
}

.table-container::-webkit-scrollbar-thumb:hover {
  background: linear-gradient(135deg, #5a6fd8, #6a4190);
}
</style>