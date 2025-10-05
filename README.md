# Home Assistant PV-EV Surplus Charging Automation

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024+-blue.svg)](https://www.home-assistant.io/)

Intelligent EV charging automation that uses only surplus energy from your photovoltaic (PV) system. Maximizes self-consumption of solar power while preventing grid import during EV charging.

## 🌟 Features

### Core Functionality
- **PV Surplus Charging**: Charges EV only with excess solar energy after house consumption
- **Automatic Phase Switching**: Dynamically switches between Monophasic and Triphasic charging based on available power
- **Dynamic Current Adjustment**: Continuously adjusts charging current (1-16A) to match available surplus
- **Dual API Support**: Compatible with both Tesla API and generic EV charger APIs

### Smart Power Management
- **Configurable Power Sharing**: Set default (50%) and max (80%) share percentages for EV
- **Battery SOC Integration**: Automatically switches to max share when house battery reaches 100%
- **Margin Allowance Logic**: Prevents frequent current oscillations with grace period counter
- **Reserved Power**: Maintains a configurable margin for battery charging

### Safety Features
- **Battery SOC Protection**: Prevents EV charging when house battery is below minimum level
- **Grid Import Block**: Stops charging if grid import is detected
- **Emergency Stop**: Automatic shutdown on critical sensor failures
- **Sensor Validation**: Validates all required sensors before operation

### Update Strategy
- **Sequential Update Logic**: Guarantees Phase → Current → Start sequence
- **Safe Phase Changes**: Always stops charging before phase mode updates
- **Update Interval**: Configurable update frequency (10-600 seconds)

## 📋 Requirements

### Home Assistant
- Home Assistant 2024.0 or newer
- Package support enabled

### Hardware Requirements
- **PV System** with power monitoring
- **Home Battery System** with SOC (State of Charge) monitoring
- **EV Charger** with:
  - Phase mode control (Monophasic/Triphasic)
  - Current control (1-16A or 8-16A depending on API)
  - Charging status reporting

### Required Sensors
The following Home Assistant sensors must be available:

| Sensor | Description | Unit |
|--------|-------------|------|
| `sensor.gw_pv` | PV production power | W |
| `sensor.gw_load` | Total house load | W |
| `sensor.gw_battery` | Battery State of Charge | % |
| `sensor.gw_from_grid` | Grid import power | W |
| `sensor.ev_charger_power` | EV charger power consumption | kW |
| `sensor.ev_charger_status` | EV charger status | State |

### Required Controls
| Entity | Description | Values |
|--------|-------------|--------|
| `select.ev_charger_mode` | Phase mode selector | Monophasic, Triphasic |
| `select.ev_charger_charging` | Charging ON/OFF | on, off |
| `number.ev_charger_set_charge_current` | Current control | 8-16A |
| `switch.tesla_model_y_charge` | Tesla charging switch (optional) | on, off |
| `number.tesla_model_y_charge_current` | Tesla current control (optional) | 1-16A |

## 🚀 Installation

### 1. Download Files
Download the following files from this repository:
- `pv_ev_surplus.yaml` - Main automation package
- `pv_ev_surplus_dashboard.yaml` - Dashboard configuration

### 2. Install Package
Copy `pv_ev_surplus.yaml` to your Home Assistant `packages` directory:

```bash
# Example path (adjust to your setup)
/config/packages/pv_ev_surplus.yaml
```

If you don't have packages enabled, add this to your `configuration.yaml`:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

### 3. Configure Entity Names
**Important:** Update all entity references in `pv_ev_surplus.yaml` to match your actual sensor/entity names.

Search and replace the following entities with your own:
- `sensor.gw_pv` → your PV power sensor
- `sensor.gw_load` → your house load sensor
- `sensor.gw_battery` → your battery SOC sensor
- `sensor.gw_from_grid` → your grid import sensor
- `sensor.ev_charger_power` → your EV charger power sensor
- `sensor.ev_charger_status` → your EV charger status sensor
- `select.ev_charger_mode` → your EV phase mode control
- `select.ev_charger_charging` → your EV charge ON/OFF control
- `number.ev_charger_set_charge_current` → your EV current control

### 4. Install Dashboard (Optional)
Add the dashboard to your Lovelace configuration:

```yaml
# In ui-lovelace.yaml or via UI
views:
  - !include pv_ev_surplus_dashboard.yaml
```

### 5. Reload Home Assistant
- Go to **Developer Tools** → **YAML**
- Click **"Check Configuration"**
- If valid, click **"Restart"** or reload **"Automations"**

## ⚙️ Configuration

### Input Helpers

All configuration is done through input helpers that appear in the Home Assistant UI:

#### Main Controls
| Setting | Default | Range | Description |
|---------|---------|-------|-------------|
| **PV-EV Surplus Automation** | OFF | - | Master ON/OFF switch |
| **Tesla Current API** | ON | - | Use Tesla API for current control (1-16A) |
| **Tesla Charge ON/OFF API** | ON | - | Use Tesla API for charging control |
| **Min Battery SOC Block** | ON | - | Enable battery SOC protection |
| **Grid Import Block** | ON | - | Enable grid import protection |

#### Timing
| Setting | Default | Range | Description |
|---------|---------|-------|-------------|
| **EV Update Interval** | 60s | 10-600s | How often to update charging parameters |
| **Phase Mode Update Delay** | 5s | 1-30s | Delay before/after phase changes |

#### Power Sharing
| Setting | Default | Range | Description |
|---------|---------|-------|-------------|
| **EV Share Percentage Default** | 50% | 0-100% | Normal surplus share for EV |
| **EV Share Percentage Max** | 80% | 0-100% | Max surplus share (at 100% battery) |
| **EV Share Percentage** | Default | Default/Max | Current selector (auto-switched) |

#### Margin & Allowance
| Setting | Default | Range | Description |
|---------|---------|-------|-------------|
| **Margin Percentage** | 20% | 0-50% | Reserved power margin percentage |
| **EV Reserved Margin Allowance** | 5 | 1-20 | Grace cycles before current decrease |

#### Current Limits
| Setting | Default | Range | Description |
|---------|---------|-------|-------------|
| **Tesla Min Charging Current** | 1A | 1-16A | Minimum for Tesla API |
| **EV Charger Min Charging Current** | 8A | 8-16A | Minimum for EV Charger API |
| **Max Monophasic Current** | 14A | 8-16A | Maximum current for single phase |
| **Min Triphasic Current** | 9A | 3-15A | Minimum current per phase (3-phase) |

#### Safety
| Setting | Default | Range | Description |
|---------|---------|-------|-------------|
| **Min House Battery SOC** | 20% | 0-100% | Minimum battery level before blocking EV charge |

## 📊 How It Works

### Power Flow Calculation
```
1. PV Surplus = PV Power - Real House Load
   (Real House Load excludes EV charger power)

2. Available Power for EV = PV Surplus × Share Percentage

3. Reserved Power = PV Surplus - Available Power for EV

4. Margin Power = Reserved Power × Margin Percentage
```

### Phase Selection Logic
```
Available Current = Available Power / 230V

IF Available Current < Min Charging Current:
    → STOP charging (insufficient power)

ELIF Available Current > Max Monophasic:
    → Use TRIPHASIC mode

ELIF Available Current ≥ Min Triphasic AND divisible by 3:
    → Use TRIPHASIC mode

ELIF Available Current ≥ Min Charging Current:
    → Use MONOPHASIC mode
```

### Current Calculation
```
Monophasic:
    Target Current = Available Current (rounded to integer, 1-16A)

Triphasic:
    Target Current = Available Current / 3 (rounded to integer, 1-16A per phase)
```

### Margin Allowance Logic
When surplus decreases and current needs to drop:

1. Calculate power deficit
2. If deficit ≤ margin power:
   - Increment grace counter
   - Keep current unchanged
3. If counter reaches max allowance:
   - Decrease current
   - Reset counter
4. On current increase or when current matches target:
   - Reset counter

This prevents rapid on/off cycling due to cloud shadows or brief load spikes.

### Update Sequence
Every update interval, the automation:

1. **Safety Checks**: Verify EV connected, battery SOC OK, no grid import
2. **Sufficient Current**: Ensure target current ≥ minimum
3. **Phase Update** (if needed): Stop charging → Update phase → Wait
4. **Current Update** (if needed): Update current (with margin logic)
5. **Start Charging**: Ensure charging is ON with correct settings

## 🐛 Debugging

### Debug Sensors
The package includes helpful debug sensors:

- **`sensor.pv_ev_debug_why_not_charging`**: Shows exactly why charging isn't active
- **`sensor.pv_ev_safety_check`**: Current safety status
- **`sensor.pv_ev_system_health`**: Sensor availability check

### Debug Notifications
When automation is active, persistent notifications show:
- Automation triggers and state
- Phase/current updates
- Margin grace periods
- Safety stop reasons

### Common Issues

**Charging not starting:**
- Check `sensor.pv_ev_debug_why_not_charging`
- Verify all required sensors are available
- Ensure sufficient current (target ≥ minimum)
- Check safety blocks aren't triggered

**Frequent on/off cycling:**
- Increase `Margin Percentage` (default 20%)
- Increase `EV Reserved Margin Allowance` (default 5 cycles)
- Increase `EV Update Interval` (default 60s)

**Phase not switching:**
- Verify `select.ev_charger_mode` options are exactly `"Monophasic"` and `"Triphasic"`
- Check `Phase Mode Update Delay` is sufficient (default 5s)

## 📈 Advanced Features

### Battery SOC Automation
When house battery reaches 100% SOC:
- Automatically switches to "Max" share percentage
- More surplus goes to EV instead of export
- Reverts to "Default" when battery drops below 100%

### API Mode Selection
**Tesla API Mode** (recommended for Tesla):
- Enables 1-16A current control
- Uses native Tesla charge switch
- More granular control at low currents

**EV Charger API Mode** (for other chargers):
- Uses 8-16A current control
- Uses charger's built-in controls
- Better compatibility with generic chargers

### Emergency Stop
Automatic emergency stop triggered by:
- Battery SOC unavailable or < 5%
- Grid import > 5000W
- EV charger status unknown/unavailable
- PV power < 0 (sensor error)

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📝 License

This project is licensed under the MIT License - see below for details:

```
MIT License

Copyright (c) 2025

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## 🙏 Acknowledgments

- Home Assistant community
- Tesla API developers
- Growatt integration contributors

## 📧 Support

If you find this project helpful, please consider starring the repository!

For issues or questions, please use the GitHub Issues page.

---

**Disclaimer:** This automation controls EV charging equipment. Always monitor initial operation and verify safety features work correctly in your specific setup. The authors assume no liability for equipment damage or incorrect operation.
