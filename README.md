// ----------------------
// Redragon M711 Cobra (16-zone) Plugin for SignalRGB
// ----------------------
export function Name() { return "Redragon M711 Cobra"; }
export function VendorId() { return [0x04D9]; }
export function ProductId() { return [0xFC62]; }
export function Publisher() { return "Piclesdd"; }
export function Type() { return "Hid"; }
export function Size() { return [8, 2]; } // 8x2 grid = 16 LEDs

// ----------------------
// LEDs e posições visuais
// ----------------------
const vLedNames = [
  "Rear Left", "Rear Right",
  "Side L1", "Side R1",
  "Side L2", "Side R2",
  "Side L3", "Side R3",
  "Side L4", "Side R4",
  "Side L5", "Side R5",
  "Front Left", "Front Right"
];
const vLedPositions = [
  [0,0],[1,0],
  [2,0],[3,0],
  [4,0],[5,0],
  [6,0],[7,0],
  [0,1],[1,1],
  [2,1],[3,1],
  [4,1],[5,1],
  [6,1],[7,1]
];
export function LedNames() { return vLedNames; }
export function LedPositions() { return vLedPositions; }

// ----------------------
// Configurações controláveis no app
// ----------------------
export function ControllableParameters() {
	return [
		{"property":"shutdownColor", "group":"lighting", "label":"Shutdown Color", "type":"color", "default":"000000"},
		{"property":"LightingMode", "group":"lighting", "label":"Lighting Mode", "type":"combobox", "values":["Canvas","Forced"], "default":"Canvas"},
		{"property":"forcedColor", "group":"lighting", "label":"Forced Color", "type":"color", "default":"009bde"},
		{"property":"effectMode", "group":"lighting", "label":"Effect Mode", "type":"combobox", "values":["Static","Breathing","Wave"], "default":"Static"}
	];
}

export function Initialize() {}
export function Shutdown() { sendColors(true); }
export function Render() { sendColors(); }

// ----------------------
// Comunicação HID
// ----------------------
function sendColors(shutdown = false) {
    const packetLength = 65;
    let packet = new Array(packetLength).fill(0x00);

    // Report ID e cabeçalho fixo (do dump)
    packet[0] = 0x1B;
    packet[1] = 0x00; packet[2] = 0x00;
    packet[3] = 0x35; packet[4] = 0x88;
    packet[5] = 0x81; packet[6] = 0x07;
    packet[7] = 0x84; packet[8] = 0xFF;

    // Escrever cores de 16 LEDs
    for (let i = 0; i < 16; i++) {
        let pos = vLedPositions[i];
        let col;

        if (shutdown) {
            col = hexToRgb(shutdownColor);
        } else if (LightingMode === "Forced") {
            col = hexToRgb(forcedColor);
        } else {
            col = device.color(pos[0], pos[1]);
        }

        // efeitos
        if (effectMode === "Breathing") {
            const breath = Math.abs(Math.sin(Date.now() / 500)) * 255;
            col = col.map(c => (c * breath) / 255);
        } else if (effectMode === "Wave") {
            const wave = (Math.sin(Date.now() / 150 + i) + 1) / 2;
            col = col.map(c => (c * wave));
        }

        // cada LED ocupa 3 bytes RGB, começando do índice 15
        const base = 15 + i * 3;
        packet[base]   = Math.round(col[0]);
        packet[base+1] = Math.round(col[1]);
        packet[base+2] = Math.round(col[2]);
    }

    device.write(packet, packetLength);
}

// ----------------------
// Auxiliares
// ----------------------
function hexToRgb(hex) {
	let result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
	return result ? [
		parseInt(result[1], 16),
		parseInt(result[2], 16),
		parseInt(result[3], 16)
	] : [0,0,0];
}

export function Validate(endpoint) {
	return endpoint.interface === 0;
}

export function ImageUrl() {
	return "https://cdn.shopify.com/s/files/1/0271/2289/3989/products/M711COBRA_01_2048x.jpg";
}
