# Assinatura de Cor IA 🎯

Sistema industrial para validação de tolerância de cor e metamerismo utilizando espectrometria e Inteligência Artificial.

## 🏗️ Arquitetura do Sistema (Modelo C4 - Container)

O diagrama abaixo ilustra a comunicação entre o hardware de borda (Light Box), o processamento backend e a interface de operação:

```mermaid
graph TD
    %% Atores
    Operador((Operador de<br/>Qualidade))

    %% Sistema (Boundary)
    subgraph Assinatura de Cor IA
        direction TB
        ESP[Hardware: ESP32 + AS7341<br/>C++ / PlatformIO]
        API[Backend & IA<br/>Node.js + Python]
        DB[(Banco de Dados<br/>PostgreSQL)]
        WEB[Dashboard Web<br/>Vue.js 3 + Tailwind]
    end

    %% Relacionamentos
    Operador -->|Posiciona peça na cabine| ESP
    Operador -->|Monitora tolerância| WEB
    ESP -->|Envia 11 canais espectrais (JSON HTTP)| API
    WEB -->|Consome alertas e métricas| API
    API <-->|Grava/Lê laudos do lote via Prisma| DB
```
