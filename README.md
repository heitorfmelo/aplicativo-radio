# 📱 Aplicativo de rádio - Multiplataforma e Aúdio simultâneo

![Status](https://img.shields.io/badge/Status-Em_Produ%C3%A7%C3%A3o-success?style=for-the-badge)
![Flutter](https://img.shields.io/badge/Flutter-SDK-02569B?style=for-the-badge&logo=flutter&logoColor=white)
![Dart](https://img.shields.io/badge/Dart-Language-0175C2?style=for-the-badge&logo=dart&logoColor=white)
![Architecture](https://img.shields.io/badge/Architecture-Provider_Pattern-003B57?style=for-the-badge)

> **Aviso:** Este é um repositório vitrine. O código-fonte primário desta aplicação é mantido em ambiente privado corporativo. Este documento descreve as decisões de engenharia, arquitetura de software e integrações nativas aplicadas no desenvolvimento deste player de rádio multiplataforma.

---

## 🎯 Visão Geral da Arquitetura

O aplicativo **Sistema Atalaia Rádios** foi desenhado para contornar os complexos desafios de streaming contínuo em *background* nos sistemas operacionais Android e iOS. O foco da arquitetura foi garantir o menor consumo de bateria possível, estabilidade de conexão (lidando com perdas de pacote) e o controle rigoroso do estado da aplicação utilizando o **Provider Pattern** (`ChangeNotifierProvider`).

A aplicação agrega múltiplas estações (como Metropolitana, Novabrasil e Atalaia 96) em uma única base de código nativamente compilada.

---

## 🧠 Engenharia e Lógica de Negócios (Core Features)

### 1. Resiliência de Streaming e Configuração Dinâmica
Sistemas de áudio ao vivo são suscetíveis a falhas de servidor. Para mitigar esse risco de negócio:
*   **Firebase Remote Config:** As URLs de *stream* não são chumbadas (hardcoded) no binário. A aplicação intercepta configurações na nuvem, permitindo que a equipe de engenharia altere a URL do servidor de áudio em tempo real (`_remoteConfig.getString('url_${radio.id}')`), sem necessidade de submeter uma nova versão às lojas (App Store/Google Play).
*   **Monitoramento de Conectividade:** Uma *Subscription* assíncrona (`connectivity_plus`) escuta ativamente o hardware do dispositivo. Em caso de queda para *offline*, o *Player* suspende imediatamente a renderização e o consumo de dados, estabilizando a *Thread* principal.

### 2. Integração Nativa e Audio Session
Um dos maiores desafios mobile é o gerenciamento de recursos em *background*:
*   **Foreground Services & Wake Locks:** Configuração estrita no `AndroidManifest.xml` (`FOREGROUND_SERVICE_MEDIA_PLAYBACK` e `WAKE_LOCK`) acoplada ao `just_audio_background`, garantindo que o Android não destrua a aplicação quando a tela for bloqueada ou a RAM estiver sob pressão.
*   **Áudio Focus (OS Level):** Integração com o `audio_session` definindo o *profile* `AudioSessionConfiguration.music()`. Isso garante que a aplicação respeite a hierarquia de áudio do SO (ex: pausar a rádio automaticamente se o usuário abrir o Spotify ou receber uma chamada telefônica).

### 3. Telemetria e Analytics em Singleton
A análise de retenção de ouvintes foi construída com um padrão `Singleton` (`AnalyticsService`) para evitar múltiplas instâncias concorrentes na RAM.
*   **Rastreio de Sessão (Stopwatch):** Em vez de apenas contar "cliques", o sistema utiliza um *Stopwatch* interno no *Controller*. Quando o usuário dá o *Stop*, o serviço loga eventos estruturados (`radio_listening_session`) enviando a duração exata da sessão em segundos/minutos para o Firebase. O envio só ocorre se a sessão durar mais de 5 segundos, purificando os dados contra cliques acidentais.
*   **Captura de Erros de Stream:** Falhas de *buffer* disparam logs automáticos de `stream_error` atrelados à estação correspondente, permitindo o monitoramento de estabilidade da rede de forma proativa.

### 4. Sincronização de Hardware e UI/UX
*   **Controle de Hardware Híbrido:** Integração bidirecional do volume. Se o usuário aperta o botão físico do celular, o UI reflete (`VolumeController`). O último volume utilizado é persistido via `SharedPreferences`, restaurando o estado exato na próxima abertura.
*   **Metadados (ICY) e API Externa:** A aplicação extrai metadados nativos injetados no fluxo de áudio da rádio (nome da música e artista) de forma assíncrona via `icyMetadataStream`. Imediatamente, realiza um *fetch* na API do iTunes (`https://itunes.apple.com/search`) convertendo a string em uma imagem de alta resolução (Capa do Álbum) `500x500`, que é injetada tanto na UI com filtros de desfoque (`ImageFilter.blur`) quanto na notificação da tela de bloqueio.

---

## 🛡️ Stack Tecnológica & Pacotes Core

*   **Framework & Linguagem:** Flutter SDK, Dart
*   **Gerência de Estado:** Provider (`ChangeNotifierProvider`)
*   **Core de Áudio:** `just_audio`, `just_audio_background`, `audio_session`
*   **Infraestrutura BaaS:** Firebase Core, Firebase Analytics, Firebase Remote Config
*   **Interface e Caching:** `cached_network_image`, `google_fonts` (Tipografia Premium Outfit), `shimmer`, `font_awesome_flutter`.

---
*Para dúvidas técnicas sobre a sincronização da *Audio Session* com o sistema operacional, a captura de metadados ICY no stream, ou a arquitetura do Singleton de Analytics, sinta-se à vontade para entrar em contato.*
