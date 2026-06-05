# 📱 Aplicativo Rádio Multiplataforma e Áudio Simultâneo

![Status](https://img.shields.io/badge/Status-Em_Produ%C3%A7%C3%A3o-success?style=for-the-badge)
![Flutter](https://img.shields.io/badge/Flutter-SDK-02569B?style=for-the-badge&logo=flutter&logoColor=white)
![Dart](https://img.shields.io/badge/Dart-Language-0175C2?style=for-the-badge&logo=dart&logoColor=white)
![Architecture](https://img.shields.io/badge/Architecture-Provider_Pattern-003B57?style=for-the-badge)

> **Aviso:** Este é um repositório vitrine. O código-fonte primário desta aplicação é mantido em ambiente privado corporativo. Este documento descreve as decisões de engenharia, a arquitetura de software e os desafios técnicos resolvidos no desenvolvimento deste player de rádio multiplataforma.

---

## 🎯 O Desafio de Engenharia

Aplicações de streaming de áudio contínuo enfrentam desafios rigorosos em dispositivos móveis, principalmente relacionados ao agressivo gerenciamento de bateria e memória (RAM) dos sistemas operacionais (Android e iOS). 

O principal objetivo arquitetural deste projeto foi construir um motor de reprodução resiliente que operasse perfeitamente em *background*, sincronizasse metadados em tempo real e sobrevivesse às interrupções de rede e do sistema operacional, mantendo uma interface fluida e reativa.

---

## 🧠 Arquitetura e Soluções Técnicas

### 1. Streaming Resiliente e Alta Disponibilidade
Servidores de rádio ao vivo podem sofrer instabilidades. Para proteger a operação e evitar interrupções prolongadas:
*   **Configuração Dinâmica em Nuvem:** A aplicação adota o modelo de *Remote Configuration*. Nenhuma URL de streaming é fixa no binário do aplicativo. Se um servidor cair, a equipe de engenharia atualiza a rota de áudio no painel de controle e o aplicativo passa a consumir a nova fonte instantaneamente, eliminando a necessidade de lançar atualizações nas lojas de aplicativos.
*   **Monitoramento de Conectividade:** A camada de rede monitora ativamente o hardware. Em caso de perda de conexão, o sistema suspende o consumo de dados e o processamento da *Thread* principal, evitando travamentos e o esgotamento da bateria do usuário.

### 2. Gerenciamento de Ciclo de Vida e Background
Garantir que a música não pare quando a tela é bloqueada exigiu integração profunda com as APIs nativas:
*   **Isolamento de Processo e Wake Locks:** Integração de serviços de primeiro plano (*Foreground Services*) e permissões de *Wake Lock* no manifesto nativo, impedindo que o coletor de lixo (Garbage Collector) do Sistema Operacional destrua o player quando a aplicação não estiver visível.
*   **Audio Focus Handling:** Implementação de regras estritas de hierarquia de áudio. O aplicativo detecta quando o usuário atende uma ligação ou abre outra mídia (como um vídeo no YouTube) e gerencia automaticamente o *ducking* (redução de volume) ou a pausa total da rádio, retornando quando o foco é recuperado.

### 3. UX Dinâmica e Sincronização de Metadados
Para enriquecer a experiência do ouvinte além do áudio puro:
*   **Processamento Assíncrono de Metadados (ICY):** O motor de áudio extrai silenciosamente os metadados embutidos no *stream* ao vivo (nome da música e artista).
*   **Integração com APIs Externas:** Imediatamente após capturar o metadado, uma requisição assíncrona é feita à API do iTunes para buscar a capa oficial do álbum em alta resolução. Essa imagem é inserida em cache e aplicada tanto na interface principal (com efeitos de *blur* dinâmicos) quanto na central de controle da tela de bloqueio do celular.

### 4. Telemetria Estruturada e Inteligência de Negócio
A coleta de dados foi projetada para entregar métricas acionáveis, e não apenas contagem de cliques:
*   **Análise de Retenção (Stopwatch):** O controlador de estado possui um cronômetro interno. Ao finalizar uma audição, o aplicativo envia a duração exata da sessão para o serviço de *Analytics*. O sistema possui um filtro que ignora sessões muito curtas (inferiores a 5 segundos) para manter os dados de retenção precisos e livres de ruído.
*   **Rastreio Proativo de Erros:** Quedas de *buffer* ou falhas de codec disparam eventos silenciosos detalhando o motivo do erro e a estação afetada, permitindo a atuação preventiva da engenharia.

---

## 🏗️ Padrões de Projeto (Design Patterns)

*   **State Management (Provider Pattern):** Toda a lógica de negócio, controle de volume, *sleep timer* e regras de *playback* foram desacopladas da interface de usuário utilizando o padrão `ChangeNotifier`. Isso garante que a UI seja apenas um reflexo do estado atual da aplicação, facilitando testes e manutenção.
*   **Singleton Pattern:** Aplicado no serviço de *Analytics* para garantir um único ponto de acesso e processamento de telemetria durante todo o ciclo de vida do aplicativo, otimizando o uso de memória.

---

## 🛠️ Stack Tecnológica

*   **Framework:** Flutter SDK
*   **Linguagem:** Dart
*   **Gerência de Estado:** Provider
*   **Integrações Nativas (Audio & SO):** Just Audio, Audio Session
*   **BaaS (Backend as a Service):** Firebase (Core, Analytics, Remote Config)
*   **Design & UI:** Google Fonts, Cached Network Image, Shimmer

---
*Este documento reflete a arquitetura de uma aplicação real em operação. Para discussões técnicas sobre a implementação da gerência de estado, integração nativa ou arquitetura do projeto, o autor está à disposição.*
