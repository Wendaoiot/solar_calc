<template>
  <div class="card">
    <section class="section">
      <h2>1. 负载（可添加多个设备）</h2>

      <!-- 设备库选择 -->
      <div class="library-row">
        <label>从设备库添加
          <div style="display:flex;gap:8px;align-items:center">
            <select v-model="selectedDeviceId">
              <option v-for="d in devices" :key="d.id" :value="d.id">{{ d.name }} — {{ d.category }} — {{ d.power_typ }} W</option>
            </select>

            <!-- 估算档位选择 -->
            <div style="display:flex;align-items:center;gap:6px;margin-left:6px">
              <label style="font-size:13px;color:#333">档位：</label>
              <select v-model="estimateMode" style="padding:6px;border-radius:6px">
                <option value="optimistic">乐观（低）</option>
                <option value="typical">典型</option>
                <option value="conservative">保守（高）</option>
              </select>
            </div>

            <button class="btn" @click="addFromLibrary">添加</button>
            <div class="source-note" style="margin-left:8px;font-size:12px;color:#666">
              数据来源：<strong v-if="devicesSource==='external'">外部 /devices.json</strong>
              <strong v-else-if="devicesSource==='bundle'">内置 (bundle)</strong>
              <span v-else>无设备库</span>
              <a href="/docs/设备库说明.html" target="_blank" rel="noopener" style="margin-left:8px;font-size:12px">说明</a>
            </div>
          </div>
        </label>
      </div>

      <div v-for="(l, i) in loads" :key="i" class="load-row">
        <div class="row">
          <label>名称
            <input v-model="l.name" placeholder="灯、摄像头 等" />
          </label>
          <label>功率 (W)
            <input type="number" v-model.number="l.power" min="0" />
          </label>
          <label>小时/天
            <input type="number" v-model.number="l.hours" min="0" step="0.1" />
          </label>
          <label>数量
            <input type="number" v-model.number="l.qty" min="1" />
          </label>
          <button class="btn small" @click="removeLoad(i)">删除</button>
        </div>
      </div>
      <button class="btn" @click="addLoad">添加空白负载</button>
    </section>
    <!-- 简单 modal 显示系统参数说明 -->
    <div v-if="showSysDoc" class="modal-overlay">
      <div class="modal">
        <h3>系统参数说明（DoD 与效率）</h3>
        <p>DoD（可放电深度）表示电池已放电容量占额定容量的比例。50% 常作为铅酸电池的保守值；典型模式下我们使用 60% 作为折中值，LiFePO4 等锂电可使用更高 DoD。</p>
        <p>逆变/系统效率：当系统为电池直供（无逆变）时效率可近似为 100%。</p>
        <div style="text-align:right;margin-top:12px">
          <button class="btn" @click="showSysDoc = false">关闭</button>
        </div>
      </div>
    </div>

    <section class="section">
      <h2>2. 系统参数</h2>
      <div class="grid">
        <label>平均有效日照小时 (h)
          <div style="display:flex;gap:8px;align-items:center">
            <input type="number" v-model.number="sunHours" min="0.1" step="0.1" />
            <select v-model="sunHoursPreset" style="padding:6px;border-radius:6px">
              <option value="south">南方（典型 5 h）</option>
              <option value="central">中部（典型 4 h）</option>
              <option value="north">北方（典型 3 h）</option>
              <option value="custom">自定义</option>
            </select>
          </div>
          <div style="font-size:12px;color:#666;margin-top:4px">
            数据来源：基于气象年均日照小时与工程经验值。提供“南方/中部/北方”典型参考值以便快速选择。
            <a href="/docs/系统参数说明.html" target="_blank" rel="noopener" style="margin-left:6px">详情</a>
          </div>
        </label>
        <label>备用天数 (天)
          <input type="number" v-model.number="daysAutonomy" min="0" />
        </label>
        <label>系统电压 (V)
          <div style="display:flex;gap:8px;align-items:center">
            <select v-model="batteryVoltagePreset" style="padding:6px;border-radius:6px">
              <option value="3.7">单串三元锂 — 3.7 V</option>
              <option value="7.4">双串三元锂 — 7.4 V</option>
              <option value="11.1">三串三元锂 — 11.1 V</option>
              <option value="3.2">单串磷酸铁锂 — 3.2 V</option>
              <option value="6.4">双串磷酸铁锂 — 6.4 V</option>
              <option value="9.6">三串磷酸铁锂 — 9.6 V</option>
              <option value="12.8">四串磷酸铁锂 — 12.8 V</option>
              <option value="6">铅酸 — 6 V</option>
              <option value="12">铅酸 — 12 V</option>
              <option value="24">常见 24 V 系统</option>
              <option value="custom">自定义 (V)</option>
            </select>
          </div>
          <div v-if="batteryVoltagePreset==='custom'" style="margin-top:8px">
            <input type="number" v-model.number="batteryVoltage" min="3" step="0.1" style="padding:8px;border:1px solid #e6e9ef;border-radius:6px;min-width:120px;width:150px" />
          </div>
          <div style="font-size:12px;color:#666;margin-top:4px">可从常用电池电压选择（列举三元锂、磷酸铁锂、铅酸常用电压），也支持自定义。</div>
        </label>

        <!-- 系统参数模式（改为分段按钮） -->
        <label>系统参数模式
          <div style="display:flex;gap:8px;align-items:center;flex-wrap:wrap">
            <div class="segmented">
              <button :class="['seg', systemMode==='optimistic' ? 'active' : '']" @click.prevent="systemMode='optimistic'">乐观</button>
              <button :class="['seg', systemMode==='typical' ? 'active' : '']" @click.prevent="systemMode='typical'">典型</button>
              <button :class="['seg', systemMode==='conservative' ? 'active' : '']" @click.prevent="systemMode='conservative'">保守</button>
            </div>
            <button class="btn small" @click.prevent="applySystemMode(systemMode)" title="将当前模式的默认值应用到字段">重置为模式默认</button>
            <div style="font-size:12px;color:#666;margin-left:6px">切换模式会设置 DoD、效率与面板衰减的默认值，之后仍可手动调整。</div>
          </div>
        </label>

        <label>电池可放电深度 DoD (%) <span class="help" title="可放电深度 (Depth of Discharge)，越大表示允许放出更多电量，但会影响寿命">?</span>
          <input type="number" v-model.number="dodPercent" min="1" max="100" />
          <div style="font-size:12px;color:#666;margin-top:4px">
            默认基于电池化学与工程经验；切换模式可快速选择推荐值。
            <a href="#" @click.prevent="showSysDoc = true" style="margin-left:6px">了解更多</a>
          </div>
        </label>

        <!-- 基础参数显示；高级参数折叠隐藏 -->
        <label>逆变/系统效率 (%) <span class="help" title="系统从电池到负载的整体效率，若无逆变可设为 100%">?</span>
          <input type="number" v-model.number="inverterEffPercent" min="10" max="100" />
        </label>

        <div class="adv-toggle-row">
          <button class="btn" @click.prevent="showAdvanced = !showAdvanced">{{ showAdvanced ? '隐藏高级参数' : '显示高级参数' }}</button>
          <div style="font-size:12px;color:#666;margin-left:8px;align-self:center">高级参数包含回收效率与面板衰减</div>
        </div>

        <div v-if="showAdvanced">
          <label>电池回收效率 (%) <span class="help" title="充放电循环中能量损失导致的效率">?</span>
            <input type="number" v-model.number="batteryEffPercent" min="10" max="100" />
          </label>

          <label>光伏系统综合系数 (面板衰减/控制器/连线) (%) <span class="help" title="综合损失系数，越高越好（表示损失越小）">?</span>
            <input type="number" v-model.number="panelDeratePercent" min="10" max="100" />
          </label>
        </div>
      </div>
    </section>

    <section class="section result">
      <h2>3. 计算结果</h2>
      <div class="result-grid">
        <div class="result-item">
          <div class="label">总日能耗（AC）</div>
          <div class="value">{{ formatWh(totalDailyAcWh) }}</div>
        </div>

        <div class="result-item">
          <div class="label">考虑逆变后的DC日能耗</div>
          <div class="value">{{ formatWh(totalDailyDcWh) }}</div>
        </div>

        <div class="result-item">
          <div class="label">建议电池容量</div>
          <div class="value">{{ formatWh(batteryCapacityWh) }} · {{ formatAh(batteryCapacityAh) }} @ {{ batteryVoltage }}V</div>
        </div>

        <div class="result-item">
          <div class="label">所需组件总功率 (Wp)</div>
          <div class="value">{{ formatW(requiredPanelWp) }}</div>
        </div>

        <!-- 面板块数与单块功率已移除：保留总功率 (Wp) 作为建议指标 -->
      </div>

      <div class="notes">
        <strong>说明：</strong>
        日能耗 = Σ(功率 W × 小时 × 数量)。
        计算假设把 AC 负载换算到 DC（除以逆变效率），电池按 DoD 可用容量计，面板按平均日照与系数计算。
      </div>
    </section>
  </div>
</template>

<script setup>
import { reactive, ref, computed, watch, onMounted } from 'vue'
import devicesBundle from '../data/devices.json'

// 负载列表（响应式）
const loads = reactive([
  { name: '示例 - 灯', power: 10, hours: 8, qty: 2 },
  { name: '示例 - 摄像头', power: 5, hours: 24, qty: 1 }
])

// 系统参数（使用 ref，template 中自动解包）
const sunHours = ref(4)
// sun hours presets: 'south'|'central'|'north'|'custom'
const sunHoursPreset = ref('central')
const sunHoursCustom = ref(4)
const daysAutonomy = ref(2)
const batteryVoltage = ref(12)
// battery voltage presets: keep a preset selector and allow custom
const batteryVoltagePreset = ref('12')
const batteryVoltageCustom = ref(12)
const dodPercent = ref(50)
const inverterEffPercent = ref(90)
const batteryEffPercent = ref(90)
const panelDeratePercent = ref(90)
// 系统参数模式：optimistic | typical | conservative
const systemMode = ref('typical')
// whether to show advanced parameters
const showAdvanced = ref(false)

function applySystemMode(mode){
  // 模式对应的默认值（可调整）
  if (mode === 'optimistic'){
    dodPercent.value = 80
    inverterEffPercent.value = 100
    batteryEffPercent.value = 95
    panelDeratePercent.value = 85
  } else if (mode === 'conservative'){
    dodPercent.value = 50
    inverterEffPercent.value = 90
    batteryEffPercent.value = 85
    panelDeratePercent.value = 70
  } else { // typical
    dodPercent.value = 60
    inverterEffPercent.value = 100
    batteryEffPercent.value = 90
    panelDeratePercent.value = 90
  }
}

onMounted(()=>{
  applySystemMode(systemMode.value)
})

watch(systemMode, (v)=> applySystemMode(v))

// 当日照参考选项改变时，设置 sunHours（除 custom）
watch(sunHoursPreset, (v)=>{
  if (v === 'south') sunHours.value = 5
  else if (v === 'central') sunHours.value = 4
  else if (v === 'north') sunHours.value = 3
  // custom: 不自动覆盖，用户可输入
})

// 电压预设变化：若不是 custom 则把对应数值赋给 batteryVoltage
watch(batteryVoltagePreset, (v)=>{
  if (v === 'custom'){
    batteryVoltage.value = Number(batteryVoltageCustom.value) || batteryVoltage.value
    return
  }
  const n = Number(v)
  if (!isNaN(n) && n > 0) batteryVoltage.value = n
})

// 若用户修改自定义值，把 batteryVoltage 跟随
watch(batteryVoltageCustom, (v)=>{
  if (batteryVoltagePreset.value === 'custom') batteryVoltage.value = Number(v) || batteryVoltage.value
})

// devices: reactive array used by template; 初始使用内置 bundle，运行时尝试从 /devices.json 拉取覆盖
const devices = ref(Array.isArray(devicesBundle) ? devicesBundle : [])
// 记录数据来源：'bundle' | 'external' | 'none'
const devicesSource = ref(devices.value.length ? 'bundle' : 'none')

// 选中的设备库 id
const selectedDeviceId = ref(devices.value.length ? devices.value[0].id : '')
// 估算档位: 'optimistic' | 'typical' | 'conservative'
const estimateMode = ref('typical')
// 显示系统参数说明 modal
const showSysDoc = ref(false)

async function loadExternalDevices(){
  try{
    const res = await fetch('/devices.json', { cache: 'no-store' })
    if (!res.ok) throw new Error('no external')
    const json = await res.json()
    if (Array.isArray(json) && json.length){
      devices.value = json
      devicesSource.value = 'external'
      selectedDeviceId.value = devices.value[0]?.id || ''
      return
    }
  }catch(e){
    // keep bundle
    devicesSource.value = devices.value.length ? 'bundle' : 'none'
  }
}

onMounted(()=>{
  // 尝试异步加载 public/devices.json（可替换）
  loadExternalDevices()
})

function addFromLibrary(){
  const d = devices.value.find(x => x.id === selectedDeviceId.value)
  if (!d) return
  // 选择档位时优先使用 d.power_min / d.power_max（若存在），否则使用相对系数
  let power = Number(d.power_typ) || 0
  if (estimateMode.value === 'conservative'){
    if (d.power_max !== undefined) power = Number(d.power_max)
    else power = Math.round(power * 1.2 * 100) / 100
  } else if (estimateMode.value === 'optimistic'){
    if (d.power_min !== undefined) power = Number(d.power_min)
    else power = Math.round(power * 0.8 * 100) / 100
  }
  loads.push({ name: d.name, power: power, hours: d.hours_typ || 0, qty: 1 })
}

function addLoad(){
  loads.push({ name:'', power:0, hours:0, qty:1 })
}
function removeLoad(i){
  loads.splice(i,1)
}

// 核心计算
const totalDailyAcWh = computed(() => {
  return loads.reduce((sum, l) => sum + (Number(l.power || 0) * Number(l.hours || 0) * Number(l.qty || 1)), 0)
})

const totalDailyDcWh = computed(() => {
  const invEffFrac = (Number(inverterEffPercent.value) / 100) || 1
  if (invEffFrac <= 0) return 0
  return totalDailyAcWh.value / invEffFrac
})

const batteryCapacityWh = computed(() => {
  const dod = (Number(dodPercent.value) / 100) || 0.5
  const battEff = (Number(batteryEffPercent.value) / 100) || 0.9
  const days = Number(daysAutonomy.value) || 0
  if (dod <= 0) return 0
  return (totalDailyDcWh.value * (days || 1)) / (dod * battEff)
})

const batteryCapacityAh = computed(() => {
  const v = Number(batteryVoltage.value) || 12
  if (v <= 0) return 0
  return batteryCapacityWh.value / v
})

const requiredPanelWp = computed(() => {
  const sun = Number(sunHours.value) || 0
  const derate = (Number(panelDeratePercent.value) / 100) || 0.75
  if (sun <= 0 || derate <= 0) return 0
  return totalDailyDcWh.value / (sun * derate)
})

// panelsNeeded removed — we keep requiredPanelWp (总 Wp) as the primary suggestion

function _trimZeros(s){
  return (''+s).replace(/\.0+$|(?<=\.[0-9]*[1-9])0+$/,'')
}
function formatWh(v){
  if (!isFinite(v) || v <= 0) return '0 Wh'
  // show kWh for large values
  if (v >= 1000) return _trimZeros((v/1000).toFixed(2)) + ' kWh'
  // for small values keep precision to avoid showing 0
  if (v < 1) return _trimZeros(v.toFixed(3)) + ' Wh'
  if (v < 10) return _trimZeros(v.toFixed(2)) + ' Wh'
  return Math.round(v) + ' Wh'
}
function formatW(v){
  if (!isFinite(v) || v <= 0) return '0 W'
  if (v < 1) return _trimZeros(v.toFixed(3)) + ' W'
  return Math.round(v) + ' Wp'
}
function formatAh(v){
  if (!isFinite(v) || v <= 0) return '0 Ah'
  if (v < 1) return _trimZeros(v.toFixed(3)) + ' Ah'
  if (v < 10) return _trimZeros(v.toFixed(2)) + ' Ah'
  return Math.round(v) + ' Ah'
}

</script>

<style scoped>
.card{background:#fff;padding:18px;border-radius:8px;box-shadow:0 6px 18px rgba(15,23,42,0.06)}
.section{margin-bottom:18px}
.load-row .row{display:flex;gap:8px;align-items:center;margin-bottom:8px}
label{display:flex;flex-direction:column;font-size:13px;color:#333}
input{padding:8px;border:1px solid #e6e9ef;border-radius:6px;min-width:80px}
.btn{background:var(--accent);color:#fff;padding:8px 12px;border-radius:6px;border:none;cursor:pointer}
.btn.small{background:#ef5350;padding:6px}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(180px,1fr));gap:10px}
.grid > label > div{display:flex;gap:8px;align-items:center;flex-wrap:wrap}
.result-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:12px}
.result-item{background:#f7f9fc;padding:12px;border-radius:8px}
.label{font-size:13px;color:#666}
.value{font-size:18px;font-weight:600;margin-top:6px}
.notes{margin-top:12px;color:#444;font-size:13px}

.segmented{display:inline-flex;border:1px solid #dfe6ef;border-radius:6px;overflow:hidden}
.segmented .seg{background:transparent;border:none;padding:6px 10px;cursor:pointer}
.segmented .seg.active{background:var(--accent);color:#fff}
.help{display:inline-block;margin-left:6px;background:#eef3ff;color:#1a3c8a;border-radius:50%;width:18px;height:18px;line-height:18px;text-align:center;font-size:12px;cursor:help}
.adv-toggle-row{display:flex;align-items:center;grid-column:1/-1;margin-top:4px}

.modal-overlay{position:fixed;inset:0;background:rgba(0,0,0,0.4);display:flex;align-items:center;justify-content:center}
.modal{background:#fff;padding:18px;border-radius:8px;max-width:600px;width:90%;box-shadow:0 10px 30px rgba(0,0,0,0.2)}

@media (max-width:520px){
  .load-row .row{flex-direction:column;align-items:stretch}
}

</style>
