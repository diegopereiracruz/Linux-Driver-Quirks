[English Version](https://github.com/diegopereiracruz/Linux-Driver-Quirks/blob/main/sdhci_debug_quirks_EN.md)

# debug_quirks

O parâmetro `sdhci.debug_quirks` no Linux é usado para **depurar ou ajustar o comportamento do driver SDHCI (Secure Digital Host Controller Interface)**, que controla interfaces de cartão SD e eMMC em sistemas Linux.

---

### 🧠 O que são "quirks"?

No contexto de drivers do kernel Linux, *quirks* são **ajustes especiais para lidar com peculiaridades de hardware**, especialmente para controladores ou dispositivos que não seguem exatamente os padrões. Esses ajustes podem ativar ou desativar recursos específicos, corrigir bugs ou melhorar a compatibilidade com certos chips.

---

### 📌 `sdhci.debug_quirks` — função

O parâmetro `sdhci.debug_quirks` permite **forçar manualmente a ativação de determinados quirks** no driver `sdhci`. Isso pode ser útil para resolver problemas com leitura/gravação em cartões SD, desempenho, inicialização de dispositivos e outros comportamentos anômalos.

Ele é geralmente usado como parâmetro de boot do kernel (via GRUB, por exemplo):

```bash
sdhci.debug_quirks=0xVALOR
```

---

### 🔢 Valores possíveis

Você define `sdhci.debug_quirks` como uma **máscara de bits** (em hexadecimal), em que cada bit ou grupo de bits ativa um determinado quirk. Esses valores variam conforme a versão do kernel, mas alguns comuns incluem:

| Valor (bit) | Quirk                                                 |
| ----------- | ----------------------------------------------------- |
| `0x01`      | Ignora o bit de estado do cartão (`NO_CARD_NO_RESET`) |
| `0x02`      | Força reset em cada inserção do cartão                |
| `0x04`      | Desativa o uso de DMA (usa PIO em vez de DMA)         |
| `0x08`      | Não faz tuning (evita ajustes finos de clock)         |
| `0x10`      | Evita acesso a registradores específicos              |
| `0x20`      | Desabilita suporte a alta velocidade (HS)             |

---

### 🔧 Como usar?

1. **Testar problemas com cartão SD:**
   Se seu cartão SD falha ao montar ou tem erros de I/O, você pode testar desabilitando recursos:

   ```bash
   sdhci.debug_quirks=0x04
   ```

2. **Alterar GRUB temporariamente:**
   No menu do GRUB, edite a linha do kernel (`linux ...`) e adicione ao final:

   ```
   sdhci.debug_quirks=0x04
   ```

3. **Persistente:**
   Edite o arquivo:

   ```bash
   sudo nano /etc/default/grub
   ```

   E adicione ao final de `GRUB_CMDLINE_LINUX_DEFAULT`:

   ```
   sdhci.debug_quirks=0x04
   ```

   Depois:

   ```bash
   sudo update-grub
   ```

---

### 📎 Dica para descobrir os valores disponíveis

Você pode procurar no código-fonte do kernel (driver `sdhci.c` ou `sdhci-pci.c`) ou usar:

```bash
modinfo sdhci
```

Ou pesquisar quirks no repositório do kernel:

```bash
grep -i quirk /usr/src/linux/drivers/mmc/host/sdhci*
```

# Valores

Aqui está um panorama mais completo dos possíveis valores (flags) que você pode usar no `sdhci.debug_quirks` (e em `sdhci.debug_quirks2`, disponível em kernels mais recentes):

---

## 🛠️ Principais flags para debug\_quirks (bits em `debug_quirks`)

Embora a lista exata possa variar conforme a versão do kernel, valores comumente encontrados em `drivers/mmc/host/sdhci.h` incluem:

* `0x01` — **SDHCI\_QUIRK\_BROKEN\_CARD\_DETECTION**: ignora o estado do cartão quando a detecção via GPIO falha.
* `0x02` — **SDHCI\_QUIRK\_NO\_POWER\_ON\_RESET**: evita reset automático ao inserir o cartão.
* `0x04` — **SDHCI\_QUIRK\_BROKEN\_DMA**: desativa DMA (usa PIO).
* `0x08` — **SDHCI\_QUIRK\_NO\_HISPD**: desliga suporte a High Speed.
* `0x10` — **SDHCI\_QUIRK\_NO\_ENDATTR\_IN\_NOPDESC**: evita acessos a certos registradores.
* `0x20` — **SDHCI\_QUIRK\_BROKEN\_TIMEOUT\_VAL**: corrige tempos de timeout errados.
* `0x40` — **SDHCI\_QUIRK\_BROKEN\_ADMA**: desativa ADMA (como sugerido no AskUbuntu) ([askubuntu.com][1], [github.com][2], [android.googlesource.com][3], [bugzilla.kernel.org][4]).

Esses flags são especialmente úteis para contornar problemas como detecção falha de cartão, timeouts, erros DMA ou falhas em velocidade alta.

---

## 🛠️ Flags para `debug_quirks2`

Introduzido em versões mais recentes do kernel, esse parâmetro permite quirks adicionais:

* `0x04` (exemplo citado): reduz de Ultra‑HS para HS .
* Outros bits (como `SDHCI_QUIRK2_NO_1_8_V`, `SDHCI_QUIRK2_SLOW_INT_CLR`, `SDHCI_QUIRK2_ALWAYS_USE_BASE_CLOCK`, etc.) controlam comportamento de voltagem, interrupções, clocks e retuning no driver ([android.googlesource.com][3]).

---

## 📌 Exemplos práticos

* `sdhci.debug_quirks=0x40` — força desativar ADMA; útil em leitores Broadcom ou com falhas DMA ([askubuntu.com][1]).
* `sdhci.debug_quirks=0x20000` — "desliga" CQE (Command Queue Engine), útil ao formatar eMMC .
* `sdhci.debug_quirks2=0x4` — reduz de Ultra‑HS para HS para compatibilidade ([askubuntu.com][1]).

---

## 🧩 Como descobrir os bits individuais?

1. Verifique o cabeçalho do driver no kernel local:

   ```bash
   grep -R "SDHCI_QUIRK_" /usr/src/linux/drivers/mmc/host/sdhci.h
   ```

2. Ou consulte a documentação do seu kernel ou da versão que estiver usando.

---

## 📝 Resumo

* **`debug_quirks`**: máscara de bits (0x…) para ativar quirks principais: detecção, reset, DMA/ADMA, velocidade, timeout, etc.
* **`debug_quirks2`**: quirks mais avançados, presentes em kernels mais recentes.
* Valores hexadecimais combinam quantos flags quiser (ex: `0x40 | 0x04 = 0x44`).

[1]: https://askubuntu.com/questions/695944/sd-card-doesnt-work-well-under-ubuntu-but-does-well-under-windows?utm_source=chatgpt.com "14.04 - SD card doesn't work well under Ubuntu, but does well ..."
[2]: https://github.com/intel/idxd-driver/blob/master/drivers/mmc/host/sdhci.c?utm_source=chatgpt.com "idxd-driver/drivers/mmc/host/sdhci.c at master - GitHub"
[3]: https://android.googlesource.com/kernel/msm/%2B/android-7.1.0_r0.2/drivers/mmc/host/sdhci.c?utm_source=chatgpt.com "drivers/mmc/host/sdhci.c - kernel/msm - Git at Google"
[4]: https://bugzilla.kernel.org/show_bug.cgi?id=73241&utm_source=chatgpt.com "73241 – SDHCI PCI driver incompatible with 14e4:16bc / Broadcom ..."
