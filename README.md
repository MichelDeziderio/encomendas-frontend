# Gestão de encomendas condominiais — especificação v1

## POC
Página estática index.html, sem backend, sem banco externo. Na Vercel, escolha Other, raiz do projeto, sem comando de build. Para teste local use um servidor HTTP (ex.: npx serve .). Use dados e etiquetas fictícios ou anonimizados.

### Teste
1. Portaria: fotografe ou selecione etiqueta, clique em Ler etiqueta. Tesseract.js executa OCR no navegador em português/inglês; texto bruto e confiança aparecem. Nome, bloco, apartamento e rastreio são sugestões heurísticas editáveis. Confirme os dados.
2. Confira setor e prazo (dois dias por padrão; úteis sem feriados ou corridos).
3. Morador: selecione o perfil demonstrativo correspondente, fotografe encomenda e confirme.
4. Inspecione IndexedDB / Auditoria: packages, queue e audit, fotos e timestamps locais.
5. Desligue a conexão com a página aberta, registre retirada, reconecte e simule confirmação; recarregue a página para verificar persistência.
6. A confirmação simulada é manual, não há servidor nem horário oficial.

### Limitações
OCR baixa biblioteca, worker e modelos de fontes externas na primeira utilização; a imagem é processada localmente. Não há garantia de extração correta. A POC não é uma PWA instalável nem garante recarga offline. Dados não são compartilhados entre aparelhos, navegadores ou origens; podem ser apagados pelo navegador. Perfis são simulados, não há autenticação nem isolamento real, e a auditoria pode ser alterada localmente. Não utilizar em produção. Testes reais de câmera e OCR pendentes.

## Escopo do MVP
SaaS multi-condomínio, PWA de autoatendimento. O porteiro recebe, fotografa etiqueta, OCR sugere nome, bloco, apartamento e rastreio opcional, funcionário confere, confirma e guarda no setor informado. O morador recebe aviso, localiza pacote sozinho, fotografa a própria encomenda e confirma recebimento. Portaria não acompanha retirada. Após prazo configurável (dias úteis/corridos, feriados e fuso do condomínio), sistema alerta funcionário autorizado para mover pacote à área geral; localização muda somente após confirmação física. Notificar morador sobre mudança.

Perfis: administrador SaaS (sem acesso rotineiro aos dados), síndico (cadastros, configuração, indicadores e auditoria), portaria (entrada e movimentação), morador (somente suas encomendas). Ativação condicionada a 100% dos aceites individuais exigidos; registrar versão do termo, titular, data e evidência. Prever novos moradores, revogação e procedimento alternativo sem prejuízo. Validar base legal e papéis controlador/operador com jurídico; consentimento não elimina demais obrigações. Finalidade exclusivamente logística, sem leads, venda, publicidade ou treinamento de IA. Minimização, retenção configurada e acesso auditado.

### Requisitos
- Cadastro de condomínio, blocos, unidades, moradores, funcionários, setores e termos versionados.
- OCR gratuito local, revisão manual obrigatória, não associar destinatário ambíguo automaticamente.
- Recebimento com foto, responsável, horário oficial e setor; prazo padrão proposto de dois dias.
- Notificações de entrada, vencimento e transferência; dashboard com pendências por bloco, retiradas, vencidas, área geral.
- Retirada independente com foto obrigatória e confirmação eletrônica do morador autenticado; comparar rastreio quando possível (complementar).
- Estados: aguardando, prazo expirado, transferida, retirada; pendente de sincronização é estado local; divergências em análise.
- Fila IndexedDB com fotos Blob, UUID por operação, retentativas com backoff, retomada ao abrir/reconectar, idempotência no backend, não apagar até confirmação de foto e dados persistidos; erros permanentes exigem resolução. PWA fechada não garante execução contínua; armazenamento pode ser perdido.
- capturedAt (relógio do dispositivo, não verificado), receivedAt (backend ao receber), persistedAt (registro de gravação confirmada). UTC, exibir America/Sao_Paulo ou fuso configurado. Registrar modo online/offline, tentativas declaradas e observadas, erros. Não inventar horário oficial offline.
- Auditoria protegida contra mudanças não autorizadas; correções por novos eventos; trilha auxilia investigação e cruzamento com câmeras existentes, não prova autoria de furto.
- Segurança física e acesso à sala a definir; alternativa para quem não possui celular.

## Arquitetura proposta de produção
Angular/PrimeNG/Signals/PWA, Tesseract.js, IndexedDB, NestJS REST, MongoDB/Mongoose, armazenamento privado R2, Web Push/VAPID, JWT/refresh e RBAC; frontend Vercel, backend Railway, CI GitHub Actions, testes Jest/Vitest e Playwright. Versões a validar. HTTPS, autorização por tenant e recurso, URLs temporárias, validação de arquivos, rate limits, backups, relógio sincronizado, observabilidade, política de retenção, testes de isolamento e cobertura alvo 80%.

Coleções: condominiums, blocks, units, users, memberships, consents, termsVersions, packages, storageLocations, withdrawals, syncOperations, auditLogs, notifications, incidents, settings.
Package: id, tenantId, residentId, blockId, unitId, trackingCode opcional, intakePhotoKey, storageLocationId, status, receivedBy, receivedAt, withdrawalDeadline, transferredAt, withdrawnAt, createdAt, updatedAt.
Withdrawal: id, tenantId, packageId, residentId, operationId, photoKey, confirmed, connectionMode, capturedAt, receivedAt, persistedAt, retryCountReported, serverAttempts. Identidade e permissões determinadas no servidor.

API proposta: POST /auth/login, /condominiums, /residents, /consents, /packages/identify, /packages, /packages/:id/withdraw, /sync/operations, /incidents; GET /condominiums/:id/settings, /consents/summary, /packages/me, /packages/expired, /sync/operations/:id, /audit-logs, /dashboard; PATCH /packages/:id/location. Validar autorização e tenant em todos os endpoints privados.

## Homologação e entregas
POC com 30–50 etiquetas anonimizadas de transportadoras distintas e teste de foto offline; definir meta de precisão após amostra. Em seguida estrutura multi-tenant/auth, onboarding e consentimento, catalogação, retirada offline, prazo e notificações, dashboard e auditoria, piloto. Estimativa preliminar anterior de 6–8 semanas em dedicação integral, sujeita a validação.
Aceite: ativação bloqueada sem todos os aceites; OCR revisável; acesso restrito; prazo e transferência corretos; foto e confirmação obrigatórias; fila persiste após reiniciar; reenvio não duplica; limpeza só após confirmação; três horários diferenciados; fuso correto; auditoria acessível ao síndico; isolamento entre tenants; conflitos em análise.
Pendências: nome comercial, retenção (90 dias apenas proposta), autorização familiar, acesso físico, revogação e novos moradores, calendário de feriados, preço por condomínio. Fora do MVP: apps nativos, armários inteligentes, fechaduras integradas, reconhecimento facial, integração com transportadoras/câmeras e IA generativa.
