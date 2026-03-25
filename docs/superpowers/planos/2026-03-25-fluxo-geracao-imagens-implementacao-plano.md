# Plano de Implementação: Fluxo de Geração de Imagens IA

## Fase 1: Setup do Ambiente (Semana 1)

### 1.1 Instalação do ComfyUI
- [ ] Clone do repositório ComfyUI
- [ ] Instalação do ComfyUI-Manager
- [ ] Configuração de ambiente Python
- [ ] Teste de inicialização básica

### 1.2 Validação de Hardware
- [ ] Teste de VRAM disponível
- [ ] Configuração de execução em GPU (AMD + ROCm ou CPU fallback)
- [ ] Teste de geração básica com SDXL

### 1.3 Instalação de Modelos Base
- [ ] Download SDXL 1.0 (base + refiner)
- [ ] Download modelo vae-sdxl
- [ ] Download modelos ControlNet (OpenPose, Depth, Canny)
- [ ] Teste de geração com prompts básicos

---

## Fase 2: Identity Layer (Semana 2)

### 2.1 Instalação de Nós de Identidade
- [ ] Instalar ComfyUI-InstantID
- [ ] Instalar ComfyUI-ipAdapter
- [ ] Configurar nós de face embedding

### 2.2 Teste de Preservação de Rosto
- [ ] Teste com 1 imagem de referência
- [ ] Teste com múltiplas referências
- [ ] Ajuste de parâmetros (weight, start/end step)
- [ ] Documentação de parâmetros ótimos

### 2.3 Biblioteca de Faces
- [ ] Criar estrutura de diretórios `/references/faces/`
- [ ] Criar script para salvar embeddings
- [ ] Teste de 5 rostos diferentes
- [ ] Validação de consistência

---

## Fase 3: Pipeline Completo (Semana 3)

### 3.1 Workflow de Pose
- [ ] Configurar ControlNet OpenPose
- [ ] Configurar ControlNet Depth
- [ ] Teste de geração com pose reference
- [ ] Ajuste de weights (pose vs prompt)

### 3.2 Workflow de Fusão
- [ ] Integrar InstantID com output do pose workflow
- [ ] Configurar IP-Adapter blending
- [ ] Teste de color matching
- [ ] Face restoration (GFPGAN/CodeFormer)

### 3.3 Workflow Completo Automatizado
- [ ] Criar workflow.json com todas as etapas
- [ ] Teste end-to-end
- [ ] Validação de qualidade (likeness + pose)
- [ ] Iterações de refinamento

---

## Fase 4: Refinamento (Semana 4)

### 4.1 Upscaling
- [ ] Instalar nó LDSR ou ESRGAN
- [ ] Teste de 2x upscaling
- [ ] Configurar 4x UltraSharp

### 4.2 Detailer
- [ ] Instalar ComfyUI-Detailer
- [ ] Teste de refinamento de face
- [ ] Teste de refinamento de mãos/corpo

### 4.3 Automação
- [ ] Criar batch_generator.py
- [ ] Criar face_library_manager.py
- [ ] Criar workflow_automator.py
- [ ] Teste de automação completa

---

## Fase 5: Validação e Documentação (Semana 5)

### 5.1 Testes de Qualidade
- [ ] Gerar 50 imagens de teste
- [ ] Avaliar taxa de sucesso (likeness + pose)
- [ ] Documentar taxa de sucesso por tipo de prompt

### 5.2 Documentação
- [ ] Documentar workflow completo
- [ ] Criar guia de uso rápido
- [ ] Documentar troubleshooting comum

### 5.3 Preparação para Próxima Fase
- [ ] Avaliar se setup atende necessidades
- [ ] Planejar transição para vídeo
- [ ] Identificar necessidade de upgrade de hardware

---

## Tarefas Técnicas Detalhadas

### Tarefa 1.1: Instalação ComfyUI
```bash
# Clone ComfyUI
git clone https://github.com/comfyanonymous/ComfyUI.git
cd ComfyUI

# Instalar dependências
pip install -r requirements.txt

# Baixar modelo base (SDXL)
# Precisa de ~6GB libre
```

### Tarefa 2.1: Instalação InstantID
```bash
# Via ComfyUI Manager ou manual
# Nós necessários:
# - ComfyUI-InstantID
# - insightface
# - onnxruntime
```

### Tarefa 3.3: Script de Automação (Python)
```python
# pseudo-code para workflow_automator.py
import websocket
import json

class ComfyUI Automator:
    def __init__(self, ws_address="127.0.0.1:8188"):
        self.ws = websocket.WebSocket()
        self.ws.connect(f"ws://{ws_address}/ws")
    
    def queue_prompt(self, workflow):
        # Enviar workflow para execução
        pass
    
    def get_result(self, prompt_id):
        # Buscar resultado
        pass
```

---

## Priorização de Tarefas

### Alta Prioridade (Primeiro)
1. Setup ComfyUI + validação GPU
2. Teste SDXL baseline
3. Setup InstantID + teste de face
4. Pipeline completo básico

### Média Prioridade
1. Biblioteca de faces
2. Upscaling + detailer
3. Scripts de automação

### Baixa Prioridade
1. Documentação
2. Troubleshooting avançado
3. Preparação para vídeo

---

## Dependências Externas

### Modelos (Download)
- SDXL 1.0 base + refiner (~6GB)
- InstantID (faceid_plus_v2.bin)
- ControlNet (openpose, depth, canny)
- VAE (sdxl_vae.safetensors)
- Upscaling (LDSR, ESRGAN)

### Libraries Python
- torch
- transformers
- diffusers
- opencv
- pillow
- websocket-client

---

## Estimativa de Tempo

| Fase | Tempo Estimado |
|------|----------------|
| Fase 1 (Setup) | 8-12 horas |
| Fase 2 (Identity) | 10-15 horas |
| Fase 3 (Pipeline) | 15-20 horas |
| Fase 4 (Refinamento) | 10-15 horas |
| Fase 5 (Validação) | 5-8 horas |
| **Total** | **~60 horas** |

---

## Riscos e Mitigações

### Risco: GPU AMD sem suporte oficial
**Mitigação:** Usar CPU como fallback temporário ou buscar alternativa (RunPod/Mac)

### Risco: VRAM insuficiente
**Mitigação:** 
- Usar modelos FP8/quantizados
- Reduzir resolution (512x768 em vez de 1024x1536)
- Batch size = 1

### Risco: Rosto não preserva likeness
**Mitigação:**
- Ajustar IP-Adapter weight
- Usar múltiplas referências
- Iterar com face restoration

---

## checkpoints de Validação

1. **Checkpoint 1:** ComfyUI rodando + SDXL gerando
2. **Checkpoint 2:** InstantID preservando rosto
3. **Checkpoint 3:** ControlNet controlando pose
4. **Checkpoint 4:** Pipeline completo funcionando
5. **Checkpoint 5:** Automação rodando batch

---

*Plano de implementação - 2026-03-25*