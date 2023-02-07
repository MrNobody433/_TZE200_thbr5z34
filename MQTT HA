const fz = require('zigbee-herdsman-converters/converters/fromZigbee');
const tz = require('zigbee-herdsman-converters/converters/toZigbee');
const exposes = require('zigbee-herdsman-converters/lib/exposes');
const reporting = require('zigbee-herdsman-converters/lib/reporting');
const extend = require('zigbee-herdsman-converters/lib/extend');
const e = exposes.presets;
const ea = exposes.access;
const tuya = require("zigbee-herdsman-converters/lib/tuya");


const fz2 = {
    awow_thermostat: {
        cluster: 'manuSpecificTuya',
        type: ['commandGetData', 'commandSetDataResponse'],
        convert: (model, msg, publish, options, meta) => {
            const dp = msg.data.dp;
            const value = tuya.getDataValue(msg.data.datatype, msg.data.data);

            switch (dp) {
            case 105:
                return {current_heating_setpoint_auto: (value / 2).toFixed(1)};
            case 16:
                return {current_heating_setpoint: (value / 2).toFixed(1)};
            case 2:
                const modes = {0: 'auto', 1: 'manual', 2: 'away'};
                var away_mode = 'OFF';
                if (value == 2) {
                    away_mode = 'ON';
                }
                return {mode: modes[value], away_mode: away_mode};

            case 24:
                return {local_temperature: (value / 10).toFixed(1)};

            case 34:
                return {battery_voltage: (value / 0.05).toFixed(1), battery: ((value / 0.05) - 2100) / 10};

            case 118:
                return {boost_time: value};

            case 30:
                return {child_lock: value ? 'LOCK' : 'UNLOCK'};

            case 106:
                return {timer: value};


            case 116:
                return {windowDetectionTemp: value / 2};
            case 117:
                return {windowDetectionTime: value};


            case 109: // mon
            case 114: // week
            case 115: // sun
                const days = {0: '???', 1: 'Monday', 2: 'Tuesday', 3: 'Wednesday', 4: 'Thursday', 5: 'Friday', 6: 'Saturday', 7: 'Sunday'};

                //  SUN 20  6:00 21  9:00 22  10:00 23  23:00 17  24:00

                // [7,  40, 24,  42, 36,  44, 40,   46, 92,   34, 96,    42,96,34,96,42,96,34]
                   1,   34, 24,  42,36,34,68,42,92,34,96,42,96,34,96,    42,96,34,96,42,96,34

                if (value[0] == 7) {

                    return {
                        program_sunday: {
                            sunday: [
                                '0h:0m ' + value[1] / 2 + '°C',
                                '' + parseInt(value[2] / 4) + 'h:' + (value[2] - parseInt(value[2])) * 15 + 'm ' + value[3] / 2 + '°C',
                                '' + parseInt(value[4] / 4) + 'h:' + (value[4] - parseInt(value[4])) * 15 + 'm ' + value[5] / 2 + '°C',
                                '' + parseInt(value[6] / 4) + 'h:' + (value[6] - parseInt(value[6])) * 15 + 'm ' + value[7] / 2 + '°C',
                                '' + parseInt(value[8] / 4) + 'h:' + (value[8] - parseInt(value[8])) * 15 + 'm ' + value[9] / 2 + '°C',
                                '' + parseInt(value[10] / 4) + 'h:' + (value[10] - parseInt(value[10])) * 15 + 'm ' + value[11] / 2 + '°C',
                            ]
                        },
                    }
                }

            default:
                meta.logger.warn(`zigbee-herdsman-converters:AwowThermostat: NOT RECOGNIZED DP #${
                    dp} with data ${JSON.stringify(msg.data)}`); // This will cause zigbee2mqtt to print similar data to what is dumped in tuya.dump.txt
            }
        },
    },
}

const tz2 = {
    awow_thermostat_lock: {
        key: ['lock'],
        convertSet: async (entity, key, value, meta) => {
            var lock = 0;
            if (value == 'ON') {
                lock = 1;
            }
            await tuya.sendDataPointRaw(entity, 30, [lock]);
        },
    },
    awow_thermostat_current_heating_setpoint_auto: {
        key: ['current_heating_setpoint_auto'],
        convertSet: async (entity, key, value, meta) => {
            const temp = parseInt(value * 2);
            await tuya.sendDataPointRaw(entity, 105, [0, 0 ,0, temp]);
        },
    },
    awow_thermostat_current_heating_setpoint: {
        key: ['current_heating_setpoint'],
        // set manual mode
        convertSet: async (entity, key, value, meta) => {
            const temp = parseInt(value * 2);
            await tuya.sendDataPointRaw(entity, 16, [0, 0 ,0, temp]);
            await tuya.sendDataPointRaw(entity, 2, [1]);
        },
    },
    awow_thermostat_current_mode: {
        key: ['mode'],
        convertSet: async (entity, key, value, meta) => {

            switch (value) {
            case 'auto':
                await tuya.sendDataPointRaw(entity, 2, [0]);
                break;
            case 'manual':
                await tuya.sendDataPointRaw(entity, 2, [1]);
                break;
            case 'away':
                await tuya.sendDataPointRaw(entity, 2, [2]);
                break;
            }

        },
    },
}


const definition = {
    fingerprint: [
        {
            modelID: 'TS0601',
            manufacturerName: '_TZE200_thbr5z34'
        },
    ],
    model: 'MS-SH4-Z',
    vendor: 'AWOW',
    description: 'Thermostatic radiator valve',
    supports: 'thermostat, temperature',
    onEvent: tuya.setTime,
    fromZigbee: [
        fz.ignore_basic_report,
        fz.tuya_data_point_dump,
        fz2.awow_thermostat,
        fz.ignore_tuya_set_time,
    ],
    toZigbee: [
        tz.tuya_data_point_test,
        tz2.awow_thermostat_current_heating_setpoint_auto,
        tz2.awow_thermostat_current_heating_setpoint,
        tz2.awow_thermostat_current_mode,
        tz2.awow_thermostat_lock,
    ],
    configure: async (device, coordinatorEndpoint, logger) => {
        const endpoint = device.getEndpoint(1);
        await reporting.bind(endpoint, coordinatorEndpoint, ['genBasic']);
    },
    exposes: [
        exposes.climate()
            .withSetpoint('current_heating_setpoint', 5, 30, 0.5, ea.STATE_SET)
            .withLocalTemperature(ea.STATE)
            .withAwayMode(),
        e.battery_voltage(), e.battery(),
        e.child_lock(),
    ],
};

module.exports = definition;
