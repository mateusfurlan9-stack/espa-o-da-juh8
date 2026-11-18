# Espaço da Juh — Agenda

Página das clientes: `/`. Painel administrativo protegido: `/admin`.

## Serviços e disponibilidade

O catálogo compartilhado de nomes, valores e durações está em `public/catalog.js`.
- Manicure premium: R$ 60,00, 60 minutos.
- Pedicure premium: R$ 80,00, 60 minutos.
- Combo manicure e pedicure premium: R$ 100,00, 115 minutos.
- Podologia: a partir de R$ 120,00; valor final combinado com a Juh.
- Reflexologia: R$ 85,00, 45 minutos.
- Os demais serviços e preços foram preservados.

Inícios em horas cheias, de quarta a sábado, das 9h às 17h. Extras às 18h e 19h trancados por padrão, liberados por data pela administradora, no fuso America/Sao_Paulo.
A cliente pode escolher vários serviços. A soma das durações considera todo o intervalo ocupado. A duração não reduz a grade: todos os serviços podem iniciar às 17h e terminar depois.

## Pedidos e Pix

Ao usar “Salvar pedido e continuar no WhatsApp”, nome, serviços, data e horário são registrados no banco como “Aguardando comprovante Pix”. O intervalo fica reservado e desaparece da seleção de outras clientes. A gravação confere conflitos no servidor e evita duplicar o mesmo pedido em novas tentativas.

Chave Pix: ju.santtos@icloud.com. A cliente paga somente 50% do valor via Pix para confirmar e envia o comprovante pelo WhatsApp. Os outros 50% são pagos pessoalmente no dia do atendimento. Para podologia com preço a partir de R$ 120,00, o valor final deve ser combinado com a Juh antes do sinal. Os valores do sinal e do saldo aparecem na agenda, na mensagem e no painel. O site não realiza nem verifica transferências bancárias automaticamente.

No painel, a administradora confere o comprovante e escolhe “Confirmar comprovante recebido”. Se a cliente desistir, é possível cancelar o pedido antes do prazo para liberar o horário. Pedidos pendentes expiram 3 horas após a criação, sem renovação ao atualizar a página. Sem confirmação manual do sinal pela Juh dentro desse prazo, a reserva é desativada automaticamente e o horário é liberado.

## Painel

Mostra nomes, serviços, datas, horários, valores e status, incluindo histórico e registros cancelados. Filtros: todos, aguardando comprovante, confirmados, bloqueios e cancelados. A consulta é exclusiva da conta autorizada. A API de disponibilidade pública nunca retorna nomes ou observações.

Também permite bloquear um ou vários intervalos, bloquear todos os horários de um dia, registrar atendimentos feitos fora do site, cancelar e restaurar registros. Não há sincronização com a InfinitePay.

O painel usa a autenticação do ChatGPT fornecida pelo Sites e confere a conta no servidor. Inicialmente a conta proprietária é autorizada pela configuração `ADMIN_BOOTSTRAP_EMAIL`. Em “Acesso da Juh” é possível transferir o painel para o e-mail da conta ChatGPT dela. A conta anterior perde acesso. O acesso externo ao site depende também da audiência configurada na hospedagem.

## Desenvolvimento e publicação

Requer Node.js 24 ou superior. Não há dependências externas para instalar.
- `npm run dev`: testes locais em http://localhost:4173/ e /admin.
- `npm test`: testes de disponibilidade, autenticação, reservas, conflitos e confirmação.
- `npm run build`: gera o Worker ESM em dist/server/index.js e embute arquivos públicos.

Edite `public/` e `worker/`, não os arquivos gerados de `dist/`. O catálogo é compartilhado entre página e servidor. As migrações de banco estão em `drizzle/` e são aplicadas na publicação pelo Sites.

A prévia local usa uma conta fictícia e um banco separado em `.preview/`. Essa pasta, senhas e credenciais não fazem parte do ZIP. Nunca publique o servidor de testes como servidor de produção. O ZIP agora inclui uma configuração própria para Vercel. Consulte PUBLICAR-NA-VERCEL.txt. O frontend da Vercel consulta o mesmo servidor e banco do site principal; o painel continua no servidor autenticado. Não exclua nem torne privado o site principal enquanto a Vercel estiver usando a agenda.

## Vercel

`vercel.json` define `node scripts/build-vercel.mjs` e saída `dist`. Esse build gera HTML/CSS/JS, sem o Worker e sem os arquivos do painel. Duas reescritas externas encaminham disponibilidade e reservas ao servidor principal. As respostas da agenda não são armazenadas em cache. As rotas /admin e /admin.html redirecionam ao painel autenticado. Nenhuma senha ou chave de acesso é incluída no frontend.

O servidor aceita reservas vindas de https://espacodajuh.vercel.app, https://espa-odajuh.vercel.app e do próprio domínio. Ao mudar o domínio público, atualize CLIENT_ORIGINS em worker/index.js e publique o backend. As URLs temporárias de preview podem consultar horários, mas só os domínios autorizados podem enviar reservas.

O botão de comprovante salva a reserva antes de navegar ao WhatsApp na mesma aba, evitando bloqueios de pop-up. Mantém um link de reabertura na página. Falhas de gravação aparecem junto do botão, sem declarar uma reserva como salva.

Correção do domínio em produção (confirmado nos registros do servidor): https://espa-odajuhh.vercel.app, com dois h no final de juhh. Este endereço está autorizado a salvar reservas. O ajuste de origem é feito no servidor compartilhado, sem exigir republicação da Vercel. Endereços anteriores permanecem autorizados.

Atualização de catálogo: esmaltação R$ 25,00, duração provisória de 30 minutos. As descrições completas e a ordem dos serviços estão em public/catalog.js. O sinal no crédito exige solicitar o link de pagamento à Juh pelo WhatsApp; não há cobrança automática de cartão neste site. Fundo rosa quase branco, mantendo detalhes rosa e dourado.

Grade fixa: 33 opções de início, 09:00, 09:20, 09:30, 09:50 ... 16:50, 17:00, para todos os serviços e combinações. Horários passados e intervalos ocupados continuam desativados. O bloqueio de dia inteiro inclui a opção das 17h. A migração 0002 preserva os registros existentes e permite término posterior às 17h.

Atualização: manicure tradicional e pedicure tradicional duram 50 minutos; premium dura 1 hora (manicure R$ 60, pedicure R$ 80); peeling R$ 350. Atendimento extra em 14, 15, 21, 22, 28 e 29 de dezembro de 2026. Outros dias extras não são abertos automaticamente.

Prazo do sinal: o servidor calcula created_at + 3 horas. Consultas da agenda, novas reservas e operações do painel desativam pedidos vencidos; as consultas também ignoram pendências vencidas por relógio do servidor. Não depende de aba aberta nem de temporizador do celular. O estado de expiração é gravado ao receber uma consulta. Confirmados e bloqueios não expiram. O prazo aparece na resposta da reserva, no WhatsApp e no painel, em horário de Brasília. A Juh deve conferir o pagamento e confirmar antes do vencimento: não há confirmação bancária automática. Restaurar ou confirmar uma pendência vencida é recusado; precisa de novo pedido.

## Clientes e bloqueios
Novos pedidos exigem nome e WhatsApp brasileiro com DDD. O painel /admin tem a aba Clientes, com histórico, busca e Bloquear / Desbloquear. O servidor normaliza o número e impede novos pedidos de números bloqueados mesmo com outro nome. Não cancela reservas existentes. O telefone é informado pela cliente, sem código de verificação, portanto outro número pode ser usado. Registros anteriores sem telefone continuam visíveis, sem bloqueio pelo nome. O atendimento manual pode receber telefone opcional. Dados e bloqueios ficam exclusivamente na API administrativa autenticada.

## Horários inteiros e extras
A grade pública mostra apenas os inícios 9h, 10h, …, 19h, sem horário de término. Os horários regulares vão até 17h. 18h e 19h ficam trancados em cada data; a Ju libera na seção Horários extras do painel e pode usar Marcar para registrar o atendimento. Liberar não supera folgas, intervalos ocupados ou conflitos pela duração do serviço. Trancar não cancela pedidos existentes. Os agendamentos anteriores, inclusive em horários fracionados, são preservados.

## Faturamento e recebimentos
A aba Faturamento mostra o dia ou mês, com recebimentos, sinais, devoluções, valor líquido e saldo a receber. A data do pagamento determina os recebimentos; a data do atendimento determina o saldo previsto. Ao confirmar um pedido novo ainda pendente, o sistema registra o sinal de 50% uma única vez. O saldo é lançado manualmente após a Ju conferir o pagamento. Registros antigos e atendimentos inseridos manualmente não são considerados pagos automaticamente. Use Registrar pagamento / devolução para registrar valores e datas passados. Cancelar não apaga recebimentos: registre uma devolução se houver. Ajustar valor total permite informar o preço final combinado (inclusive podologia e serviços manuais), sem baixar do total já recebido. Não há integração bancária.
