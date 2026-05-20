# Applause - Guia de Teste do Protótipo

## Sobre o Protótipo

Este protótipo implementa o mecanismo de transição fluida entre animações Lottie com a seguinte lógica:

1. **Inicialização**: Animação IDLE entra em loop contínuo
2. **Detecção de Input**: Usuário digita "SIM" no chat
3. **Transição Fluida**: Quando o loop IDLE termina naturalmente, dispara automaticamente a animação WALK
4. **Conclusão**: WALK executa uma única vez e retorna ao loop IDLE
5. **Repetição**: O ciclo pode se repetir indefinidamente

## Fluxo de Animação

```
┌─────────────────────────────────────────────────────────────┐
│                  APPLAUSE ANIMATION FLOW                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  [IDLE LOOP] ──────→ Idle completa seu ciclo ──────→        │
│      ↑                        │                    ↓         │
│      │                        │              [WALK 1x]       │
│      │                        └──────────────────┬─→ Completa│
│      │                                           │           │
│      └───────────────────────────────────────────┘           │
│                                                              │
│  Evento "SIM" marcado durante IDLE:                         │
│  • IDLE continua em loop                                    │
│  • Ao fim do loop, WALK é disparado automaticamente        │
│  • Transição é FLUIDA (sem pulo ou corte)                  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## Arquivos do Protótipo

### 1. `applause_prototype_offline.html` ⭐ (RECOMENDADO)

Versão que permite carregar animações manualmente via console.

**Como abrir:**
1. Abra o arquivo HTML em seu navegador (Chrome, Firefox, Safari, Edge)
2. Abra o Console do Navegador (F12 ou Cmd+Option+I no Mac)
3. Cole o seguinte código no console:

```javascript
// Carrega ambas as animações e as injeta no protótipo
Promise.all([
  fetch('caminho/para/0_idle.json').then(r => r.json()),
  fetch('caminho/para/1_action_walk.json').then(r => r.json())
]).then(([idle, walk]) => {
  window.setAnimations(idle, walk);
  console.log('✓ Animações carregadas com sucesso!');
});
```

**Ou, se os arquivos estiverem no mesmo diretório:**

```javascript
Promise.all([
  fetch('0_idle.json').then(r => r.json()),
  fetch('1_action_walk.json').then(r => r.json())
]).then(([idle, walk]) => {
  window.setAnimations(idle, walk);
});
```

### 2. `applause_prototype_final.html`

Versão que tenta carregar animações automaticamente via fetch.
- Útil se os arquivos JSON estiverem em um servidor ou caminho acessível via HTTP
- Menos manual que a versão offline

## Como Testar

### Teste Básico

1. Abra o protótipo em seu navegador
2. Você verá:
   - **Lado esquerdo**: Canvas com animação (grande área roxa)
   - **Lado direito**: Chat (branco)
   - **Mensagem bot**: "Você completou a missão?"

3. No chat, digite: `SIM`
4. Pressione Enter ou clique em "Enviar"

### Fluxo Esperado

```
[Chat] Você completou a missão?
[User] SIM
[Animation] IDLE continua loopando
[Animation] Idle completa seu ciclo → WALK dispara automaticamente
[Animation] WALK executa uma única vez
[Animation] WALK completa → retorna ao IDLE em loop
```

### O Que Observar

✅ **SUCESSO:**
- Animação IDLE começa a loopear continuamente
- Ao digitar "SIM", a animação continua sem mudança
- Quando o IDLE terminar seu ciclo, WALK dispara suavemente
- WALK executa uma única vez (sem loop)
- Após WALK, retorna ao IDLE sem pulo ou corte visual

❌ **PROBLEMAS:**
- Se houver pulo ou corte entre IDLE e WALK → verificar `autoTransition.delay`
- Se WALK loopear infinitamente → verificar propriedade `loop` das animações
- Se animação não iniciar → verificar se JSON foi carregado (console)
- Se animação travar → verificar se JSON é válido

## Informações Técnicas

### Animações Fornecidas

**0_idle.json:**
- Nome: `0_idle`
- Duração: 44 frames @ 12fps ≈ 3.67 segundos
- Tipo: Loop contínuo
- Conteúdo: Personagem em pose relaxada com movimentos sutis (olhos piscam, respira)

**1_action_walk.json:**
- Nome: `1_action_walk`
- Duração: 25 frames @ 12fps ≈ 2.08 segundos
- Tipo: Single play (não loop)
- Conteúdo: Personagem caminha horizontalmente (posição X muda de 489.5 → 412.5)

### Configuração de Transição

O protótipo usa a seguinte estrutura de `autoTransition`:

```javascript
animationManager.play('idle', {
  loop: true,
  autoTransition: {
    name: 'walk',
    delay: 0,  // Sem delay entre conclusão e próxima animação
    options: {
      loop: false,
      autoTransition: {
        name: 'idle',
        delay: 0,
        options: { loop: true }
      }
    }
  }
});
```

### Console Debug

Abra o Console do navegador (F12) para ver logs detalhados:

```
[AnimationManager] ✓ Animações carregadas
  - Idle: 44 frames @ 12fps (loop)
  - Walk: 25 frames @ 12fps (single)
[AnimationManager] ▶ idle (∞ loop)
[AnimationManager] ✓ idle completado
[AnimationManager] ▶ walk (1× play)
[ChatManager] ✓ Palavra-chave "SIM" detectada
[AnimationManager] ✓ walk completado
[AnimationManager] ▶ idle (∞ loop)
```

## Troubleshooting

### Problema: "Animações não carregam"

**Solução:**
1. Verifique se os arquivos JSON existem e estão acessíveis
2. Verifique o console do navegador (F12) para erros
3. Tente caminho absoluto na fetch:
   ```javascript
   fetch('file:///caminho/completo/0_idle.json')
   ```

### Problema: "Pulo entre animações"

**Solução:**
- A transição usa `autoTransition.delay: 0` para instantaneidade
- Se houver pulo visual, pode ser necessário ajustar o timing
- Verifique se ambas as animações têm o mesmo `viewBox` (954x1080)

### Problema: "WALK não para (fica em loop)"

**Solução:**
- Verifique se `1_action_walk.json` tem `op: 25` (op = out point, duração)
- Certifique-se de que a opção `loop: false` está sendo passada

### Problema: "Nada acontece ao digitar SIM"

**Solução:**
1. Verifique se as animações foram carregadas (console deve mostrar logs)
2. Digite exatamente "SIM" (maiúsculas)
3. Verifique se há erros no console do navegador

## Contato & Próximas Etapas

Este protótipo valida:
- ✓ Carregamento de animações Lottie
- ✓ Sistema de chat com detecção de keywords
- ✓ Transição fluida entre estados de animação
- ✓ AutoTransition sem frame gaps

**Próximos passos sugeridos:**
1. Refinamento visual das transições se necessário
2. Integração com servidor backend (Applause API)
3. Adicionar mais estados de animação
4. Implementar sistema de respostas dinâmicas
5. Otimização para dispositivos móveis

---

**Versão:** 1.0  
**Data:** 2026-05-20  
**Status:** ✓ Pronto para teste
