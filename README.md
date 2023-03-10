# Dashboard app

Dashboard application, connected to a node-red backend with a websocket service.

## Installation

Clone the repo then run

```bash
npm install
npm run dev
```

## References

-   https://vuejs.org/guide/introduction.html
-   https://apexcharts.com/
-   https://developer.mozilla.org/fr/docs/Web/API/WebSockets_API

## Node-Red setup

```json
[
    { "id": "50da04b3.af25fc", "type": "websocket out", "z": "d3310dac90d08ad6", "name": "", "server": "985ecbc7.67a138", "client": "", "x": 720, "y": 520, "wires": [] },
    { "id": "45dbf990.ba2408", "type": "function", "z": "d3310dac90d08ad6", "name": "Mock data", "func": "msg.payload = {\n    time: new Date().getTime(),\n    data_line: Math.floor(Math.random()*1000),\n    data_area1: 100+Math.floor(Math.random() * 500),\n    data_area2: 50+Math.floor(Math.random() * 200),\n    data_angle: 70 + Math.floor(Math.random() * 20)\n}\nreturn msg;", "outputs": 1, "noerr": 0, "initialize": "", "finalize": "", "libs": [], "x": 438, "y": 520, "wires": [["50da04b3.af25fc"]] },
    { "id": "eccc8bc2.133378", "type": "websocket in", "z": "d3310dac90d08ad6", "name": "", "server": "985ecbc7.67a138", "client": "", "x": 457, "y": 572, "wires": [["9adfff59.652"]] },
    { "id": "9adfff59.652", "type": "debug", "z": "d3310dac90d08ad6", "name": "", "active": true, "console": "false", "complete": "false", "x": 647, "y": 572, "wires": [] },
    { "id": "43fa5a0ac67fd6f2", "type": "inject", "z": "d3310dac90d08ad6", "name": "", "props": [{ "p": "payload" }, { "p": "topic", "vt": "str" }], "repeat": "0.5", "crontab": "", "once": true, "onceDelay": 0.1, "topic": "", "payload": "", "payloadType": "date", "x": 230, "y": 520, "wires": [["45dbf990.ba2408"]] },
    { "id": "985ecbc7.67a138", "type": "websocket-listener", "z": "d3310dac90d08ad6", "path": "/ws/simple", "wholemsg": "false" }
]
```
