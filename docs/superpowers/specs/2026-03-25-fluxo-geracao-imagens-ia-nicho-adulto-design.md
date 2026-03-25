# Design: Fluxo de Geração de Imagens IA no Nicho Adulto

## Visão Geral do Projeto

**Objetivo:** Criar fluxo automatizado de geração de imagens com rostos likeness + pose/corpo controlados, rodando localmente em GPU de 8GB VRAM.

**Problema atual do usuário:**
- Meses testando ferramentas manualmente
- IA "delira" - perde identidade do rosto durante geração
- Dificuldade com controle de pose + identidade simultaneamente

**Restrições Técnicas:**
- RTX 6600 8GB VRAM (AMD)
- Fully local (sem cloud para geração)
- Orçamento inicial: zero

---

## 2. Arquitetura Técnica

### Stack Principal
- **Frontend de automação:** ComfyUI (Node-based, ideal para fluxo)
- **Modelos de imagem:** Stable Diffusion XL (FP8) + Stable Diffusion 1.5
- **Controle de pose:** ControlNet OpenPose + Depth
- **Preservação de identidade:** InstantID + IP-Adapter

### Filosofia do Fluxo
> "80% da qualidade vem de múltiplos passes, não de um modelo giant"

O sistema não tenta fazer tudo em uma geração. Em vez disso, fragmenta o problema em etapas menores e iterativas.

---

## 3. Fluxo de Geração (Detalhado)

### Etapa 1: Geração do Rosto (Identity Layer)
```
Input: Referência de rosto (1-3 imagens)
Processo:
  1. InstantID carrega embedding do rosto
  2. Face ID Plus extrai landmarks faciais
  3. Output: Face embedding + landmarks
Output: Face embedding para próximas etapas
```

**Por quê InstantID?**
- Funciona bem em VRAM limitada (8GB)
- Preserva likeness melhor que IP-Adapter puro
- Suporta múltiplas referências para consistência

### Etapa 2: Geração do Corpo/Pose (Composition Layer)
```
Input: Prompt de corpo + ControlNet pose reference
Processo:
  1. ControlNet OpenPose detecta pose da referência
  2. ControlNet Depth adiciona profundidade
  3. SDXL gera base com pose controlada
  4. LowVAE optimizado para velocidade
Output: Imagem base com pose definida
```

**Por quê SDXL em vez de Flux?**
- SDXL é mais estável em GPUs limitadas
- Mais fácil de customizar com ControlNet
- Modelos FP8 cabem em 8GB

### Etapa 3: Fusão de Identidade (Blending Layer)
```
Input: Imagem base + Face embedding
Processo:
  1. IP-Adapter aplica rosto na imagem base
  2. Face restoration (GFPGAN/CodeFormer)
  3. Color correction para matching
Output: Imagem final com rosto likeness + pose controlada
```

### Etapa 4: Refinamento (Enhancement Layer)
```
Input: Imagem fundida
Processo:
  1. 2x upscaling (ESRGAN ou 4x UltraSharp)
  2. Detailer (face, hands, body)
  3. Final color grade
Output: Imagem final de alta qualidade
```

---

## 4. Pipeline Completo

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   ROSTO     │    │   POSE      │    │   FUSÃO     │    │  REFINO     │
│             │    │             │    │             │    │             │
│ InstantID   │───►│ ControlNet  │───►│ IP-Adapter  │───►│ Upscale +   │
│ Face ID     │    │ OpenPose    │    │ Face Rest.  │    │ Detailer    │
│             │    │ SDXL Base   │    │ Color Match │    │             │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

---

## 5. Biblioteca de References

**Estratégia de reuse:**
- Manter banco de 10-20 rostos testados e validados
- Cada rosto salvo com seus parâmetros ótimos (CFG, steps, sampler)
- Reutilizar rostos consistentes em vez de gerar novos

**Estrutura de armazenamento:**
```
/references
  /faces
    /face_001
      - embedding.json
      - sample_01.png
      - params.json
    /face_002
      ...
```

---

## 6. Componentes do ComfyUI

### Nós essenciais a instalar:
1. **ComfyUI-Manager** - Gerenciamento de custom nodes
2. **ComfyUI- InstantID** - Identity preservation
3. **ComfyUI ControlNet** - Pose/depth control
4. **ComfyUI-ipAdapter** - Face blending
5. **ComfyUI-LDSR** - Upscaling
6. **ComfyUI-FaceRestoration** - GFPGAN/CodeFormer
7. **ComfyUI-AdvancedRefiner** - Multi-pass refinement

### Workflow template:
O sistema deve ter um workflow.json reutilizável que:
- Permite input de imagem de rosto
- Permite input de pose reference
- Permite input de prompt
- Executa todas as etapas automaticamente
- Salva outputs formatados

---

## 7. Automação e Consistência

### Scripts de automação (Python):
1. **batch_generator.py**
   - Gera múltiplas variações de uma configuração
   - Mantém consistência usando mesma seed base

2. **face_library_manager.py**
   - Gerencia biblioteca de rostos validados
   - Tracking de parâmetros de sucesso

3. **workflow_automator.py**
   - Executa workflow completo via API do ComfyUI
   - Handle de erros e retries automáticos

### Prevenção de "delírio" da IA:
- Usar prompts estruturados comemphasis
- Limitar randomização via seed固定
- Multiplicidade de passes (não one-shot)
- Checkpoint de validação: rosto reconhecível?

---

## 8. Validação e Testing

### Critérios de qualidade:
1. **Likeness:** Rosto reconhecível comparando com referência?
2. **Pose:** Pose segue a referência ControlNet?
3. **Anatomia:** Mãos, dedos, corpo sem distorções?
4. **Consistência:** Múltiplas gerações com mesmo rosto são consistentes?

### Processo de validação:
- Gerar 10 variações de cada configuração
- Avaliar manualmente cada uma
- Documentar parâmetros que funcionam
- Iterar e refinar

---

## 9. Próximos Passos (Ordem de Implementação)

### Fase 1: Setup (Semana 1)
1. Instalar ComfyUI + Manager
2. Testar modelos baseline (SDXL + ControlNet)
3. Validar que 8GB consegue rodar workflow básico

### Fase 2: Identity Layer (Semana 2)
1. Instalar InstantID + IP-Adapter
2. Testar preservação de rosto
3. Criar biblioteca de referências

### Fase 3: Pipeline Completo (Semana 3)
1. Integrar todas as camadas
2. Criar workflow automatizado
3. Testar consistência end-to-end

### Fase 4: Refinamento (Semana 4)
1. Upscaling e detailer
2. Scripts de automação
3. Documentar parâmetros de sucesso

### Fase 5: Expansão (Próxima fase)
1. Adicionar geração de vídeo
2. Escalar biblioteca de rostos
3. Automação completa

---

## 10. Riscos e Mitigações

| Risco | Mitigação |
|-------|-----------|
| VRAM insuficiente | Usar modelos FP8/quantizados, batch size 1 |
| Rostos não consistent | Usar biblioteca de faces testadas |
| Pose não segue reference | Ajustar ControlNet weights |
| Resultado看起来fake | Multi-pass refinement + upscaling |

---

## Resumo Final

**Abordagem:** Pipeline iterativo com separação de responsabilidades (face vs body) + fusão posterior

**Stack:** ComfyUI + InstantID + ControlNet + SDXL FP8

**Princípio:** Consistência através de múltiplos passes controlados, não one-shot com modelo giant

**Resultado esperado:** Imagens com rosto reconhecível + pose controlada, rodando em 8GB VRAM

---

*Documento criado durante brainstorming IA DO JOB*
*Data: 2026-03-25*