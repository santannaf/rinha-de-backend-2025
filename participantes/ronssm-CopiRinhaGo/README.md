## Observação sobre código gerado por IA

Todo o projeto foi desenvolvido inteiramente pelo GitHub Copilot, utilizando o modelo GPT-4.1, apenas com direcionamentos feitos por humano. Não houve validações ou alterações manuais de código: todas as implementações, revisões e correções foram realizadas exclusivamente pela IA, seguindo as instruções fornecidas.

## Overview

Este projeto implementa um backend para o desafio Rinha de Backend 2025, intermediando pagamentos e fornecendo resumo dos processamentos, com lógica robusta de fallback, health-check e conformidade total com as regras do desafio.

**Participante:** Ronaldo Santana  
**LinkedIn:** [ronaldo-santana](https://www.linkedin.com/in/ronaldo-santana/)  
**Repositório:** [github.com/ronssm/CopiRinhaGo](https://github.com/ronssm/CopiRinhaGo)

## Architecture

- **Linguagem:** Go
- **Banco de Dados:** PostgreSQL
- **Balanceador de carga:** Nginx
- **Conteinerização:** Docker Compose
- **Limites de recursos:** 1.5 CPUs e 350MB de memória entre todos os serviços
  - backend1: 0.65 CPUs, 110MB RAM
  - backend2: 0.65 CPUs, 110MB RAM
  - nginx: 0.1 CPUs, 20MB RAM
  - db: 0.1 CPUs, 110MB RAM
- **Endpoints:**
  - `POST /payments`: Intermedia pagamentos, valida UUID, escolhe o melhor Payment Processor, faz fallback e registra transações de forma síncrona, retornando HTTP 202 Accepted apenas após persistência garantida. Unicidade de `correlationId` é garantida exclusivamente pelo banco de dados.
  - `GET /payments-summary`: Retorna resumo dos pagamentos processados por processor, com suporte a filtros `from`/`to`

## Setup

1. Instale Docker e Docker Compose.
2. Clone este repositório:  
   `git clone https://github.com/ronssm/CopiRinhaGo`
3. O projeto já inclui um arquivo `.gitignore` para evitar que binários, arquivos temporários, dependências e configs locais sejam enviados ao repositório ou à submissão.
4. O projeto também inclui um arquivo `.dockerignore` para garantir que arquivos desnecessários não sejam copiados para a imagem Docker durante o build, tornando a imagem mais leve e segura.
5. Suba os Payment Processors primeiro (veja instruções do desafio).
6. O nível de log dos serviços pode ser controlado via variável de ambiente `LOG_LEVEL` no `docker-compose.yml` (exemplo: DEBUG, INFO, ERROR).

7. Para rodar localmente, basta executar:

   ```sh
   docker-compose up
   ```

   O compose já está configurado para usar a imagem publicada no Docker Hub: `ronssm/copirinhago:latest`.

   Para publicar uma nova versão da imagem no Docker Hub:

   ```sh
   docker build -t ronssm/copirinhago:latest .
   docker push ronssm/copirinhago:latest
   ```

8. Acesse os endpoints via `http://localhost:9999`.

## Conformidade com o Desafio

- Duas instâncias do backend atrás do Nginx (balanceamento de carga)
- Uso da rede `payment-processor` para integração
- Não incluir código fonte/logs na submissão
- Arquivos obrigatórios: `docker-compose.yml`, `info.json`, `README.md`, scripts SQL

## Tecnologias

- Go
- PostgreSQL
- Nginx
- Docker Compose

## Como funciona

- O endpoint de health-check é cacheado por 5s para evitar erro 429
  - Pagamentos são sempre tentados no Default (menor taxa), com fallback automático
- O endpoint `/payments` processa pagamentos de forma síncrona, respondendo apenas após persistência garantida, para máxima consistência e performance sob carga
- Todos os pagamentos são registrados com o processor usado para garantir consistência
- Validação de UUID para evitar duplicidade
- Unicidade de `correlationId` garantida apenas pelo banco de dados (sem consulta prévia)

## Recomendações de Performance

- **Pool de conexões do banco:** Ajuste `SetMaxOpenConns` e `SetMaxIdleConns` em `db/postgres.go` para valores como 30/15, conforme recursos disponíveis.
- **HTTP Client com keep-alive:** Use um client HTTP global com keep-alive para chamadas aos payment-processors, substituindo `http.DefaultClient` por um client customizado.
- **Nginx:** Aumente `worker_processes`, buffers e reduza `proxy_read_timeout` para 2s em `nginx.conf`.
- **Monitore recursos:** Ajuste limites de CPU/memória no `docker-compose.yml` conforme necessário.
- **Reduza logs em produção:** Use nível INFO ou ERROR para evitar I/O desnecessário.

## Troubleshooting

- Se aparecer erro de build Go, verifique se todos os arquivos `.go` começam com declaração de package e código válido
- Se aparecer erro de `correlationId already used`, significa que o UUID já foi processado

## License

MIT

## Changelog

- 2025-07-24 14:00: Estrutura inicial com modelo de pagamento, handlers, lógica de banco, Docker Compose, Nginx e SQL
- 2025-07-24 15:00: Adicionado cache de health-check, fallback e endpoint de resumo
- 2025-07-24 15:30: Limpeza de docker-compose.yml e nginx.conf para conformidade
- 2025-07-24 16:00: Atualização do README com setup, arquitetura e detalhes de conformidade
- 2025-07-24 16:30: Dockerfile finalizado para backend Go
- 2025-07-24 16:45: Removido código duplicado dos arquivos Go e configs
- 2025-07-24 17:00: Revisão final e checagem de conformidade para submissão
- 2025-07-24 18:00: Estrutura Go corrigida, erros de build resolvidos, lógica de pagamento aprimorada
- 2025-07-24 19:00: Validação de UUID, unicidade de correlationId, tratamento de erros de banco e limites de recursos aplicados
- 2025-07-24 20:00: Dados do participante e links atualizados para submissão
- 2025-07-24 20:15: Adicionado `.gitignore` para evitar envio de arquivos desnecessários ao repositório
- 2025-07-24 20:20: Adicionado `.dockerignore` para garantir builds Docker limpos e seguros
- 2025-07-24 20:30: Ajustes finais de documentação e instruções para submissão
- 2025-07-24 20:45: Ajustados limites de recursos dos containers no docker-compose.yml para conformidade (1.5 CPUs, 350MB RAM no total, detalhado por serviço)
  -- 2025-07-24 21:00: Endpoint /payments agora processa pagamentos de forma síncrona, respondendo apenas após persistência garantida, para máxima consistência e performance sob carga.
  2025-07-24 21:15: Otimizações técnicas finais:
- Tabela UNLOGGED: pagamentos são registrados em tabela UNLOGGED para acelerar inserts e reduzir I/O de WAL.
- Índice composto: índice composto adicionado para acelerar consultas de resumo e filtros por período.
- Autovacuum: autovacuum ativado e ajustado para performance sob alta carga.
- Tuning de buffers do Nginx: buffers aumentados para suportar respostas grandes e evitar erros de proxy.
- TTL do health-check: cache do endpoint de health-check ajustado para 5s, evitando erro 429 sob carga.
- Ordem correta do ALTER TABLE: comandos de ALTER TABLE movidos após a criação da tabela no DDL.
- `.gitignore` e `.dockerignore` revisados para garantir builds limpos e submissão sem arquivos desnecessários.

- 2025-07-25 21:00: Compose atualizado para usar imagem Docker Hub `ronssm/copirinhago:latest` e instruções de build/push adicionadas ao README
