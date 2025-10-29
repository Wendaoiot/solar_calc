![主程序界面](./1.png)

# 太阳能与电池计算器（前端小工具）

演示地址：
http://wendaoiot.com/solar_calc/

这是一个基于 Vue 3 + Vite 的前端小工具，用于估算户外设备所需的太阳能板功率与电池容量。无需后端，响应式，可在 PC 与手机浏览器访问。

快速上手（Windows PowerShell）：

```powershell
# 1. 安装依赖
npm install

# 2. 启动开发服务器
npm run dev

# 在浏览器打开 http://localhost:5173（vite 默认端口）
```

说明：
- 在页面中添加或编辑负载（功率 W、每天运行小时数、数量）。
- 填写日照小时、备用天数、系统电压、DoD(可放电深度)等参数。
- 结果包含：总日能耗、建议电池容量（Wh 与 Ah）、需用光伏总功率（Wp）以及建议面板块数。

公式（简化、供参考）：
- 日能耗（Wh）= Σ(功率 W × 小时 × 数量)
- DC 日能耗 = AC 日能耗 / 逆变效率
- 建议电池能量（Wh）= (DC 日能耗 × 备用天数) / (DoD × 电池效率)
- 电池容量 (Ah) = 建议电池能量 (Wh) / 电压 (V)
- 所需光伏功率 (Wp) = DC 日能耗 / (日照小时 × 系统综合系数)

如果你希望我把界面改成暗色主题、加入每个负载的启停时间表或支持 CSV 导入/导出，请告诉我下一步要求，我可以继续实现。


#markdown
# Solar & Battery Calculator (Frontend Tool)

**Live Demo:**  
http://wendaoiot.com/solar_calc/

A Vue 3 + Vite based frontend tool for estimating required solar panel power and battery capacity for outdoor equipment. No backend required, fully responsive, accessible on both PC and mobile browsers.

# Quick Start (Windows PowerShell)

```powershell
# 1. Install dependencies
npm install

# 2. Start development server
npm run dev

# Open http://localhost:5173 in your browser (default Vite port)
Instructions
Add or edit loads (power in W, daily operating hours, quantity) on the page
```

Fill in parameters:

Peak sun hours

Backup days

System voltage

DoD (Depth of Discharge)

Other system parameters

Results include:

Total daily energy consumption

Recommended battery capacity (Wh and Ah)

Required total PV power (Wp)

Recommended number of panels

Formulas (Simplified)
text
Daily Energy (Wh) = Σ(Power W × Hours × Quantity)
DC Daily Energy = AC Daily Energy / Inverter Efficiency
Recommended Battery Energy (Wh) = (DC Daily Energy × Backup Days) / (DoD × Battery Efficiency)
Battery Capacity (Ah) = Recommended Battery Energy (Wh) / Voltage (V)
Required PV Power (Wp) = DC Daily Energy / (Peak Sun Hours × System Composite Factor)
Future Enhancements
If you'd like me to implement any of the following features, please let me know:

Dark theme interface

Start/stop schedules for each load

CSV import/export functionality

Additional customization options

text