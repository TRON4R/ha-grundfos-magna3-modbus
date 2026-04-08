# Entity List - Grundfos MAGNA3 (English)
# 100 active, 13 commented out


## Modbus Sensors

MAGNA3 Software Version HI (Raw)  ->  sensor.magna3_software_version_hi_raw
MAGNA3 Software Version LO (Raw)  ->  sensor.magna3_software_version_lo_raw
MAGNA3 Software Date DayMonth (Raw)  ->  sensor.magna3_software_date_daymonth_raw
MAGNA3 Software Date Year (Raw)  ->  sensor.magna3_software_date_year_raw
MAGNA3 Control Register (00101 Raw)  ->  sensor.magna3_control_register_00101_raw
MAGNA3 Control Mode Request (00102)  ->  sensor.magna3_control_mode_request_00102
MAGNA3 Operation Mode Request (00103)  ->  sensor.magna3_operation_mode_request_00103
MAGNA3 Setpoint Request (00104)  ->  sensor.magna3_setpoint_request_00104
# MAGNA3 Relay Control (00105 Raw)  ->  sensor.magna3_relay_control_00105_raw
MAGNA3 Max Flow Limit Setpoint (00106)  ->  sensor.magna3_max_flow_limit_setpoint_00106
MAGNA3 Status Bits (Raw)  ->  sensor.magna3_status_bits_raw
MAGNA3 Process Feedback  ->  sensor.magna3_process_feedback
MAGNA3 Control Mode Code  ->  sensor.magna3_control_mode_code
MAGNA3 Operation Mode Code  ->  sensor.magna3_operation_mode_code
MAGNA3 Alarm Code  ->  sensor.magna3_alarm_code
MAGNA3 Warning Code  ->  sensor.magna3_warning_code
MAGNA3 Bearing Service (Raw)  ->  sensor.magna3_bearing_service_raw
MAGNA3 Drive State Code  ->  sensor.magna3_drive_state_code
MAGNA3 Feedback Sensor Unit  ->  sensor.magna3_feedback_sensor_unit
MAGNA3 Feedback Sensor Min  ->  sensor.magna3_feedback_sensor_min
MAGNA3 Feedback Sensor Max  ->  sensor.magna3_feedback_sensor_max
MAGNA3 Nominal Frequency  ->  sensor.magna3_nominal_frequency
MAGNA3 Min Frequency  ->  sensor.magna3_min_frequency
MAGNA3 Max Frequency  ->  sensor.magna3_max_frequency
MAGNA3 Setpoint Range Min  ->  sensor.magna3_setpoint_range_min
MAGNA3 Setpoint Range Max  ->  sensor.magna3_setpoint_range_max
MAGNA3 Flow Estimation State  ->  sensor.magna3_flow_estimation_state
MAGNA3 Kp Actual  ->  sensor.magna3_kp_actual
MAGNA3 Ti Actual  ->  sensor.magna3_ti_actual
MAGNA3 Direct Control  ->  sensor.magna3_direct_control
MAGNA3 Control Source Code  ->  sensor.magna3_control_source_code
MAGNA3 Head Pressure  ->  sensor.magna3_head_pressure
MAGNA3 Volume Flow  ->  sensor.magna3_volume_flow
MAGNA3 Relative Performance  ->  sensor.magna3_relative_performance
MAGNA3 Speed  ->  sensor.magna3_speed
MAGNA3 Frequency  ->  sensor.magna3_frequency
MAGNA3 Digital Input  ->  sensor.magna3_digital_input
MAGNA3 Digital Output  ->  sensor.magna3_digital_output
MAGNA3 Actual Setpoint  ->  sensor.magna3_actual_setpoint
MAGNA3 Motor Current  ->  sensor.magna3_motor_current
MAGNA3 DC Link Voltage  ->  sensor.magna3_dc_link_voltage
MAGNA3 Power Consumption  ->  sensor.magna3_power_consumption
MAGNA3 Electronic Temp  ->  sensor.magna3_electronic_temp
MAGNA3 Pump Liquid Temp  ->  sensor.magna3_pump_liquid_temp
MAGNA3 Specific Energy Consumption  ->  sensor.magna3_specific_energy_consumption
MAGNA3 Operation Time  ->  sensor.magna3_operation_time
MAGNA3 Total Powered Time  ->  sensor.magna3_total_powered_time
MAGNA3 Energy Consumption  ->  sensor.magna3_energy_consumption
MAGNA3 Number of Starts  ->  sensor.magna3_number_of_starts
MAGNA3 User Setpoint  ->  sensor.magna3_user_setpoint
MAGNA3 Diff Pressure  ->  sensor.magna3_diff_pressure
MAGNA3 Pump Unix RTC  ->  sensor.magna3_pump_unix_rtc
MAGNA3 Max Flow Limit  ->  sensor.magna3_max_flow_limit
# MAGNA3 Heat Energy  ->  sensor.magna3_heat_energy
# MAGNA3 Heat Power  ->  sensor.magna3_heat_power
# MAGNA3 Heat Diff Temp  ->  sensor.magna3_heat_diff_temp
MAGNA3 Pumped Volume  ->  sensor.magna3_pumped_volume
MAGNA3 Pumped Volume (Direction 2)  ->  sensor.magna3_pumped_volume_direction_2
# MAGNA3 Remote Pressure 1  ->  sensor.magna3_remote_pressure_1
# MAGNA3 Remote Temperature 1  ->  sensor.magna3_remote_temperature_1

## Template Binary Sensors

MAGNA3 Low Flow Stop  ->  binary_sensor.magna3_low_flow_stop
MAGNA3 Copy to Local  ->  binary_sensor.magna3_copy_to_local
MAGNA3 FlowLimit Active  ->  binary_sensor.magna3_flowlimit_active
MAGNA3 Running  ->  binary_sensor.magna3_running
MAGNA3 Remote Mode  ->  binary_sensor.magna3_remote_mode
MAGNA3 Powered On  ->  binary_sensor.magna3_powered_on
MAGNA3 Alarm  ->  binary_sensor.magna3_alarm
MAGNA3 Warning  ->  binary_sensor.magna3_warning
MAGNA3 Forced to Local  ->  binary_sensor.magna3_forced_to_local
MAGNA3 At Max Speed  ->  binary_sensor.magna3_at_max_speed
MAGNA3 At Min Speed  ->  binary_sensor.magna3_at_min_speed

## Template Sensors

MAGNA3 Software Version  ->  sensor.magna3_software_version
MAGNA3 Control Mode  ->  sensor.magna3_control_mode
MAGNA3 Operation Mode  ->  sensor.magna3_operation_mode
MAGNA3 Alarm Text  ->  sensor.magna3_alarm_text
MAGNA3 Energy Cost  ->  sensor.magna3_energy_cost
MAGNA3 Efficiency  ->  sensor.magna3_efficiency
MAGNA3 Actual Setpoint (m)  ->  sensor.magna3_actual_setpoint_m
MAGNA3 Pump RTC  ->  sensor.magna3_pump_rtc
MAGNA3 Flow Estimation State Text  ->  sensor.magna3_flow_estimation_state_text
MAGNA3 Control Source  ->  sensor.magna3_control_source
MAGNA3 Months To Bearing Service  ->  sensor.magna3_months_to_bearing_service
MAGNA3 Bearing Service Type  ->  sensor.magna3_bearing_service_type
MAGNA3 Drive State  ->  sensor.magna3_drive_state
MAGNA3 Flow Rate L/h  ->  sensor.magna3_flow_rate_l_h

## Input Numbers

input_number.magna3_setpoint_percent
input_number.magna3_setpoint_m
input_number.magna3_control_mode
# input_number.magna3_max_flow_limit
# input_number.magna3_pid_kp
# input_number.magna3_pid_ti
# input_number.magna3_pid_direct

## Scripts

script.magna3_set_setpoint_percent
script.magna3_set_setpoint_m
script.magna3_set_control_mode
script.magna3_start_remote
script.magna3_stop_remote
script.magna3_to_local
script.magna3_reset_alarm
script.magna3_enable_autoadapt
script.magna3_enable_flowadapt
script.magna3_enable_proportional
script.magna3_set_max_flow_limit
# script.magna3_set_pid_kp
# script.magna3_set_pid_ti
# script.magna3_set_pid_direct

## Automations

MAGNA3: Modbus Watchdog  ->  automation.magna3_modbus_watchdog
MAGNA3: Alarm Notification  ->  automation.magna3_alarm_notification
MAGNA3: High Current Warning  ->  automation.magna3_high_current_warning
MAGNA3: Sync Setpoint %  ->  automation.magna3_sync_setpoint
MAGNA3: Sync Setpoint m  ->  automation.magna3_sync_setpoint_m
MAGNA3: Sync Setpoints on Start  ->  automation.magna3_sync_setpoints_on_start
MAGNA3: Update Both Sliders from Pump Value  ->  automation.magna3_update_both_sliders_from_pump_value
