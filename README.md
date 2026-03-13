# 🔬 PCB Flow — Motherboard Simulator

> Simulação interativa 2D de uma placa-mãe ATX com jornada visual do dado pelo hardware — do VRM ao I/O.

![SVG](https://img.shields.io/badge/SVG-pure--2D-00ff88?style=flat-square&logo=svg)
![HTML5](https://img.shields.io/badge/HTML5-single--file-E34F26?style=flat-square&logo=html5&logoColor=white)
![Zero deps](https://img.shields.io/badge/deps-zero-00ffe0?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-cc44ff?style=flat-square)

---

## 🎯 Sobre o Projeto

**PCB Flow** é uma visualização educativa 2D de uma placa-mãe que simula a jornada completa de um dado pelo hardware — do VRM fornecendo tensão ao CPU, passando por DDR5, PCIe 5.0, chipset e chegando ao NVMe ou porta I/O.

O board é renderizado em SVG puro com estética de PCB real: FR4 verde-escuro, trilhas de cobre em camadas, vias BGA, componentes com footprints precisos e silk-screen técnico. Pacotes de dados animados percorrem os barramentos em tempo real, com zoom/pan livre.

---

## ✨ Funcionalidades

### 🟢 Board PCB Ultra-detalhada
- Placa FR4 com grid de trilhas em 2 escalas (sinal e plano de potência)
- Campo de vias BGA sob o soquete AM5 do CPU
- Roteamento Manhattan com elbows a 90°
- Keepout line tracejada e silk-screen técnico com revisão, série e fabricante
- Furos de montagem com anel de cobre e via stitching

### 📦 Componentes com Precisão de Footprint

| Componente | Detalhes visuais |
|---|---|
| **CPU · Ryzen 9 9950X** | IHS metálico, área do die, labels CCD0/CCD1/IOD, array de pinos LGA1718 |
| **RAM DDR5-6400** | 8 chips DRAM no stick, gold fingers, chip SPD EEPROM |
| **GPU · PCIe 5.0 x16** | Die + 5 chips GDDR6X + slot PCIe com pinos individuais + conector 12V |
| **NVMe SSD · M.2 2280** | Controller + 3 chips NAND + conector M.2 B+M key |
| **Chipset B650E** | Array BGA visível + marking do die + pads de soldagem |
| **I/O Panel** | USB-A codificado por cor (azul/laranja/USB4), Ethernet, 3 jacks de áudio |
| **VRM · 16+2 fases** | 4 indutores com enrolamentos, 6 MOSFETs DPAK, PWM RAA229131, capacitores bulk |

### 🔌 Periféricos e Conectores Reais
- Conector **ATX 24-pin** com pinos individuais
- Conector **EPS 8-pin** (alimentação CPU)
- 4 portas **SATA 6G** com conector em L
- 4 **fan headers** (CPU_FAN, CPU_OPT, SYS_FAN×2)
- Chip **BIOS SOIC-8** (128 Mb SPI)
- 4 **POST LEDs** (CPU / DRAM / VGA / BOOT)

### ⚡ Jornada do Dado em 8 Etapas

| Etapa | Barramento | Largura de banda |
|---|---|---|
| 1. Alimentação VRM | PWR Rail 1.1V | 16+2 fases · 90A/fase |
| 2. Fetch — Busca | DDR5 IMC | CL36 · 6400 MT/s |
| 3. Leitura DDR5 | DDR5 Dual-Channel | 102 GB/s |
| 4. DMA → GPU | PCIe 5.0 x16 | 64 GB/s |
| 5. Renderização GPU | GDDR6X intern | 1 TB/s |
| 6. CPU → Chipset | PCIe 4.0 ×4 (DMI) | 8 GB/s |
| 7. Escrita NVMe | PCIe 4.0 x4 | 7.3 GB/s |
| 8. Saída I/O | USB 3.2 / 2.5GbE | 20 Gbps |

---

## 🖱️ Navegação e Controles

### Zoom e Pan
| Ação | Mouse / Touch |
|---|---|
| Zoom in/out | Scroll do mouse (centrado no cursor) |
| Mover o board | Clique + arrastar |
| Pinch zoom | Dois dedos (mobile) |
| Resetar visão | Botão `⊹ RESET` |

### Botões do HUD
| Botão | Função |
|---|---|
| `⏸ PAUSAR` | Congela os pacotes animados |
| `◎ AUTO` | Auto-tour: avança etapa a cada 4.5s |
| `▶ PRÓXIMO` | Avança manualmente para próxima etapa |
| `＋ / － ZOOM` | Zoom incremental via botão |

### Interações diretas
- **Clique em qualquer componente** — salta para a etapa da jornada correspondente e exibe descrição técnica
- **Hover sobre componente** — tooltip com specs técnicas completas

---

## 🚀 Como Usar

### Requisitos

Nenhuma dependência. Qualquer browser moderno com suporte a SVG.

| Browser | Versão mínima |
|---|---|
| Chrome / Edge | 80+ |
| Firefox | 75+ |
| Safari | 13.1+ |

### Executar localmente

```bash
# Clone o repositório
git clone https://github.com/seu-usuario/pcb-flow.git
cd pcb-flow

# Abrir direto no browser (sem servidor necessário)
open index.html

# Ou servidor local simples
npx serve .
python3 -m http.server 8080
```

> Funciona também via `file://` diretamente — sem build step, sem npm, sem dependências.

---

## 🗂️ Estrutura

```
pcb-flow/
└── index.html    # Aplicação completa — single file HTML + SVG + JS
```

Projeto intencionalmente **single-file**: todo o SVG, CSS, JS e lógica de animação em um único arquivo HTML autocontido.

---

## 🏗️ Arquitetura Técnica

### Stack
- **SVG puro** via `document.createElementNS` — zero bibliotecas gráficas
- **requestAnimationFrame** para animação dos pacotes
- **CSS transform** com `translate + scale` para pan/zoom de alta performance
- **wheel + touch events** para controle gestual

### Sistema de Pacotes
Pacotes são elementos `<circle>` criados dinamicamente e movidos frame-a-frame por interpolação linear entre waypoints do barramento (roteamento Manhattan). Cada pacote carrega um trail de 5 círculos com alpha decrescente.

### Sistema de Zoom/Pan
```
world.style.transform = `translate(${tx}px, ${ty}px) scale(${scale})`
```
O zoom é centralizado no cursor: ao receber um evento `wheel`, a posição do mouse é usada como ponto de ancoragem para recalcular `tx` e `ty` antes de aplicar o novo `scale`.

### Camadas SVG (ordem de renderização)
```
1. Board (FR4 base + grids)
2. Decorative traces + via fields
3. Bus traces (barramentos animados)
4. Components (footprints + inner detail)
5. Labels (silk-screen)
6. Packet layer (animações — sempre no topo)
```

---

## 📚 Conceitos de Hardware Abordados

- Arquitetura AM5: IMC (controlador de memória) integrado no die do CPU
- DDR5 dual-channel e como a largura de banda se combina
- Root Complex PCIe: lanes de alta velocidade saem direto do CPU, sem hub
- DMA (Direct Memory Access): transferência sem intervenção da CPU
- DMI (Direct Media Interface): link CPU ↔ chipset
- VRM (Voltage Regulator Module): conversão e estabilização de tensão
- NVMe over PCIe: protocolo de fila para storage de alta velocidade
- Diferença entre USB 3.2 Gen2 (10 Gbps), Gen2×2 (20 Gbps) e USB4 (40 Gbps)

---

## 🤝 Contribuindo

```bash
git checkout -b feature/nova-feature
git commit -m "feat: adiciona componente X"
git push origin feature/nova-feature
```

Ideias para expansão:
- [ ] Simulador de falhas — clique para "queimar" um componente
- [ ] Modo boot — sequência POST animada
- [ ] Zoom semântico — detalhes extras aparecem conforme aumenta o zoom
- [ ] Diagrama de blocos do die do CPU (cores, cache, IMC)
- [ ] Modo comparativo — Intel vs AMD

---

## 📄 Licença

MIT — veja [`LICENSE`](LICENSE) para detalhes.

---

<div align="center">

Feito com SVG puro · zero dependências · single file

</div>
