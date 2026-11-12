# DroidSpec

**DroidSpec** é um app Android nativo de monitoramento de hardware, inspirado no HWiNFO para PC. Exibe informações em tempo real sobre CPU, GPU, memória, bateria, sensores, rede e muito mais — sem necessidade de root.

---

## Funcionalidades

| Aba | Descrição |
|---|---|
| **Painel** | Visão geral: CPU, RAM, bateria, rede, armazenamento, tela e apps |
| **CPU** | Frequência por núcleo em tempo real, modelo, arquitetura, ABI |
| **Memória** | RAM usada/disponível, heap JVM, armazenamento interno/externo |
| **Bateria** | Nível, temperatura, tensão, corrente (API 21+), status de carga |
| **Tela/GPU** | Resolução, densidade, taxa de atualização, OpenGL renderer/vendor/extensões |
| **Sensores** | Lista completa de sensores com leituras ao vivo |
| **Rede** | Tipo de conexão, SSID, BSSID, sinal Wi-Fi, operadora, endereços IP |
| **Dispositivo** | Fabricante, modelo, Android, build, kernel, uptime |
| **Câmera** | Câmeras disponíveis, resoluções, formatos de saída, flash, estabilização |
| **Processos** | Lista de processos em execução com uso de memória PSS |
| **Bluetooth** | Escaneamento de dispositivos próximos com RSSI e classe de dispositivo |
| **Testes** | Teste de lanterna, vibração, microfone e efeitos hápticos |
| **Térmica** | Temperatura por zona térmica via `/sys/class/thermal` |
| **Benchmark** | Teste de desempenho completo: CPU (single/multi-core), RAM e armazenamento |

---

## Requisitos

- Android **5.0+** (API 21)
- Nenhum root necessário
- Permissões solicitadas em tempo de execução quando necessário

## Permissões

| Permissão | Uso |
|---|---|
| `CAMERA` | Enumeração das câmeras |
| `RECORD_AUDIO` | Teste de microfone |
| `VIBRATE` | Teste de vibração |
| `BLUETOOTH_SCAN` / `BLUETOOTH_CONNECT` | Scanner Bluetooth (API 31+) |
| `ACCESS_FINE_LOCATION` | Scanner Bluetooth (API < 31) |
| `PACKAGE_USAGE_STATS` | Contagem de processos |
| `WAKE_LOCK` | Mantém atualização de sensores ativa |

---

## Stack técnica

- **Linguagem:** Kotlin
- **UI:** Jetpack Compose + Material 3
- **Arquitetura:** MVVM com `AndroidViewModel` + `StateFlow`
- **Concorrência:** Kotlin Coroutines (`Dispatchers.IO` para leituras de hardware)
- **Sensores:** Buffering com publicação a cada 300ms (evita flood de StateFlow)
- **minSdk:** 21 | **targetSdk:** 35

---

## Build

```bash
# Debug
./gradlew assembleDebug

# Release
./gradlew assembleRelease
```

APK gerado em: `app/build/outputs/apk/`

---

## Instalar via ADB

```bash
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

---

## Estrutura do projeto

```
app/src/main/java/com/hwmobile/app/
├── MainActivity.kt
├── data/
│   └── CpuReader.kt          # Leitura de /sys/devices/system/cpu/
├── ui/
│   ├── components/           # InfoCard, DonutChart, StatusProgressBar
│   ├── navigation/           # AppNavigation (tab row)
│   ├── screens/              # Uma screen por aba
│   └── theme/                # MaterialTheme, cores, tipografia
└── viewmodel/
    ├── SensorsViewModel.kt   # Buffered sensor readings
    └── NetworkViewModel.kt   # Network state polling
```

---

## Otimizações de desempenho

- **SensorsViewModel:** buffer interno com `synchronized`, publicação via StateFlow a cada 300ms (evita 100+ atualizações/segundo)
- **SensorsScreen:** `LazyColumn` com `key` estável — só renderiza sensores visíveis
- **ProcessesScreen:** `getProcessMemoryInfo` em batch único no `Dispatchers.IO`; evitado em API 31+ (retorna 0 por restrição de privacidade)
- **DashboardScreen:** `getInstalledPackages` cacheado por 30s; leituras no `Dispatchers.IO`; frequência da CPU atualizada a cada 500ms
- **NetworkViewModel / MemoryScreen:** todas as operações de I/O no `Dispatchers.IO`
- **Release build:** R8 + `isShrinkResources = true` (reduz APK significativamente)

---

## Licença

Projeto privado — uso interno.
