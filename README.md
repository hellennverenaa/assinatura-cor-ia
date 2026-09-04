# Assinatura de Cor IA 🎯

Sistema de espectrometria industrial e inteligência artificial aplicado ao controle de qualidade de materiais. O projeto substitui o julgamento visual subjetivo por assinaturas espectrais quantitativas, monitorando desvios de tonalidade, metamerismo e estabilidade de lotes em tempo real.

---

## 📊 Fluxo Operacional de Inspeção

O fluxograma abaixo apresenta o processo operacional desde o posicionamento da amostra até a persistência dos dados e a geração de indicadores estatísticos:

```mermaid
flowchart TD
    A([Início: Amostra posicionada na Cabine de Luz]) --> B[Sensor AS7341 realiza leitura dos 11 canais espectrais]
    B --> C[ESP32 serializa grandezas em JSON e transmite via Wi-Fi]
    C --> D[Backend Node.js recebe payload da leitura]
    D --> E[Motor de IA: Cálculo de Delta E e Análise Espectral]

    E --> F{Cor dentro da tolerância do lote?}
    F -- Sim --> G[Classificação: APROVADO]
    F -- Não --> H[Classificação: REPROVADO / Alerta de Desvio]

    G --> I[(Persistência no PostgreSQL via TypeORM)]
    H --> I

    I --> J[Processamento de Estatísticas: Desvio Padrão e Estabilidade]
    J --> K[Atualização do Dashboard Vue.js em Tempo Real]
    K --> L([Fim: Operador visualiza laudo e histórico de conformidade])
```

---

## 🏗️ Arquitetura do Sistema (Modelo C4)

### Nível 1: Diagrama de Contexto (System Context)

Fronteiras operacionais do sistema com usuários e sistemas corporativos.

```mermaid
graph TD
    Operador([Operador de Qualidade])
    Sistema[Sistema Assinatura de Cor IA]
    ERP[(ERP / Sistema Fabril)]

    Operador -->|Insere amostra e acompanha leituras| Sistema
    Sistema -->|Dispara alertas de inconformidade e tendências| Operador
    Sistema -->|Registra laudos técnicos e validação de insumo| ERP
```

---

### Nível 2: Diagrama de Contêineres (Containers)

Estrutura dos serviços, banco de dados e aplicações do ecossistema.

```mermaid
graph TD
    User([Operador de Qualidade])

    subgraph CoreSystem [Sistema Assinatura de Cor IA]
        direction TB
        ESP[Hardware de Borda: ESP32 + AS7341<br/>Firmware C++ / PlatformIO]
        SPA[Frontend Dashboard<br/>Vue.js 3 + Tailwind CSS]
        API[Backend API REST<br/>Node.js + TypeScript + TypeORM]
        ML[Motor Analítico & IA<br/>Python / Scikit-Learn]
        DB[(Banco Relacional<br/>PostgreSQL)]
    end

    User -->|Posiciona peça na cabine de inspeção| ESP
    User -->|Acompanha análises e gráficos em tempo real| SPA
    ESP -->|Transmite leitura de 11 canais via HTTP/JSON| API
    SPA -->|Consulta métricas, histórico e relatórios| API
    API -->|Envia leitura para inferência de tolerância| ML
    ML -->|Retorna conformidade e desvio espectral| API
    API -->|Persiste leituras brutas, laudos e métricas| DB
```

---

### Nível 3: Diagrama de Componentes (Backend API)

Organização interna dos módulos do serviço de aplicação.

```mermaid
graph TD
    subgraph BackendContainer [Backend API - Node.js / TypeORM]
        direction TB
        Controller[Inspection Controller]
        Service[Tolerance Evaluation Service]
        StatsService[Statistics & Batch Analysis Service]
        MLBridge[Python ML Execution Bridge]
        Repository[TypeORM Repositories]
        Entities[TypeORM Entities / Data Models]
    end

    Controller -->|Encaminha payload JSON| Service
    Service -->|Dispara inferência espectral| MLBridge
    Service -->|Atualiza métricas do lote| StatsService
    Service -->|Grava auditoria| Repository
    StatsService -->|Recupera histórico do lote| Repository
    Repository -->|Mapeia dados relacionais| Entities
```

---

## 📦 Lista de Materiais de Eletrônica (BOM)

A montagem aproveita a **cabine física e iluminação industrial já existentes**, demandando apenas os módulos de sensoriamento e controle:

| Componente              | Especificação Técnica                         | Função no Projeto                                             |
| :---------------------- | :-------------------------------------------- | :------------------------------------------------------------ |
| **Microcontrolador**    | ESP32 DevKit NodeMCU (USB-C, 30/38 pinos)     | Aquisição I2C, conversão JSON e envio via rede Wi-Fi          |
| **Sensor Espectral**    | AMS OSRAM AS7341 (Breakout STEMMA QT / Qwiic) | Espectrometria óptica de 11 canais (visível e NIR)            |
| **Cabo de Conexão**     | Cabo JST-SH 4 pinos para jumpers macho        | Interligação direta entre sensor e microcontrolador sem solda |
| **Fonte / Alimentação** | Fonte USB 5V 2A DC com cabo USB-C             | Alimentação elétrica dedicada para o microcontrolador         |

---

## 🔌 Pinagem e Interface Elétrica (I2C)

A comunicação física opera em nível lógico nativo de 3.3V:

| Pino AS7341 (STEMMA QT) | Pino ESP32 (GPIO) | Descrição do Sinal        | Nível Lógico |
| :---------------------- | :---------------- | :------------------------ | :----------- |
| **VIN / VCC**           | **3V3**           | Tensão de alimentação     | 3.3V DC      |
| **GND**                 | **GND**           | Referência de terra comum | 0V           |
| **SDA**                 | **GPIO 21 (D21)** | Linha serial de dados I2C | 3.3V         |
| **SCL**                 | **GPIO 22 (D22)** | Linha serial de clock I2C | 3.3V         |

---

## 💻 Stack Tecnológica

- **Hardware e Firmware:** ESP32 NodeMCU, Sensor AS7341, C++ / FreeRTOS via PlatformIO.
- **Backend:** Node.js, TypeScript, Express, TypeORM.
- **Motor Analítico & IA:** Python (NumPy, Scikit-Learn para modelos de tolerância dinâmica e cálculo de $\Delta E$).
- **Banco de Dados:** PostgreSQL (registro de leituras espectrais brutas, laudos e lotes).
- **Frontend:** Vue.js 3 (Composition API), Tailwind CSS (painel operacional, gráficos de dispersão e controle estatístico de processo).

---

## 📂 Estrutura de Diretórios (Monorepo)

```text
assinatura-cor-ia/
├── docs/                     # Diagramas C4, esquemáticos e manuais de calibração
├── firmware/                 # Código-fonte C++ do ESP32 (PlatformIO)
├── backend/                  # API REST Node.js com TypeScript e TypeORM
│   └── src/
│       ├── controllers/      # Controladores de rotas HTTP
│       ├── database/         # Configurações de conexão e migrations
│       ├── entities/         # Modelos de dados (Leitura, Inspecao, Lote)
│       └── services/         # Regras de negócio, estatística e integração
├── ml-service/               # Scripts Python de inferência espectral e tolerância
├── frontend/                 # Interface Web Vue.js 3 com painel em tempo real
└── README.md                 # Documentação unificada do projeto
```
