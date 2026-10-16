# Kairos — loja própria (Next.js + Supabase + Mercado Pago + Melhor Envio)

Loja de camisetas com painel para cadastrar produtos (com upload de fotos), frete calculado por CEP,
pagamento por **Pix** (QR Code na própria página) e **cartão** pelo Mercado Pago, com confirmação automática.

## Como funciona

| Parte | O que usa |
|---|---|
| Site e painel | Next.js 15 (App Router) |
| Produtos, pedidos e fotos | Supabase (Postgres + Storage) |
| Pix e cartão | Mercado Pago (Pix direto na página, cartão no Checkout Pro) |
| Frete | Melhor Envio (Correios, Jadlog etc.) |

Preço, estoque e frete são sempre recalculados no servidor. O navegador nunca define valores.
O pedido só vira "Pago" depois que o servidor confirma o pagamento direto na API do Mercado Pago.

## 1. Rodar no seu computador

```bash
npm install
cp .env.example .env.local     # preencha os valores (passos abaixo)
npm run dev                    # http://localhost:3000
```

## 2. Supabase (banco e fotos)

1. Crie um projeto em supabase.com.
2. Abra **SQL Editor**, cole o conteúdo de `supabase/schema.sql` e rode. Isso cria as tabelas e a pasta pública de fotos (`products`).
3. Em **Project Settings > API**, copie a *Project URL* e a chave *service_role* para `SUPABASE_URL` e `SUPABASE_SERVICE_ROLE_KEY`.
   A service_role só existe no servidor. Nunca coloque em variável `NEXT_PUBLIC_`.

## 3. Mercado Pago

1. Em mercadopago.com.br/developers, crie uma aplicação (Checkout Pro + Pagamentos online).
2. Copie o **Access Token** para `MP_ACCESS_TOKEN`. Comece com o token de teste.
3. Depois de publicar o site, em **Webhooks** da aplicação cadastre a URL
   `https://SEU-DOMINIO/api/webhooks/mercadopago`, marcando o evento **Pagamentos**.
   Copie a **assinatura secreta** para `MP_WEBHOOK_SECRET`.
4. Para ir ao vivo, troque pelo Access Token de produção.

Sem HTTPS (localhost) o webhook não chega. A página do pedido confere o Pix a cada 5 segundos
e, na volta do cartão, confirma na hora, então dá para testar o fluxo mesmo assim.

## 4. Melhor Envio (frete) — opcional

Por enquanto a loja roda só como pré-venda: com `NEXT_PUBLIC_SHIPPING_ENABLED=false` (padrão do `.env.example`)
o site não calcula nem cobra frete, o checkout mostra "A combinar" e você pode pular esta etapa.
Quando for enviar pelos Correios, troque para `true` e siga abaixo.

1. Crie conta em melhorenvio.com.br e gere um token em **Configurações > Tokens** (permissão *shipping-calculate*).
2. `MELHOR_ENVIO_TOKEN` = token, `MELHOR_ENVIO_EMAIL` = seu e-mail, `STORE_ORIGIN_CEP` = CEP de onde você despacha.
3. `MELHOR_ENVIO_SANDBOX=true` usa o ambiente de testes (o token do sandbox é diferente do de produção).
4. Cada produto tem peso e dimensões da embalagem no painel. O padrão (250 g, 30x25x4 cm) serve para camiseta dobrada.
5. Opcional: `FREE_SHIPPING_MIN_CENTS=29900` dá frete grátis (na opção mais barata) a partir de R$ 299.

A etiqueta você compra no próprio painel do Melhor Envio. Depois, no `/admin`, marque o pedido como
**Enviado** e cole o código de rastreio. Ele aparece na página do pedido do cliente.

## 5. Painel (/admin)

Defina `ADMIN_PASSWORD` e `SESSION_SECRET` (texto aleatório de 32+ caracteres) e acesse `/admin`.
Lá você cadastra produtos (fotos, preço, tamanhos, estoque, pré-venda), vê pedidos, muda o status e informa o rastreio.

Pré-venda: marque a opção no produto. Ele aceita pedidos sem estoque e mostra o selo "Pré-venda".

## 6. Publicar

Qualquer hospedagem que rode Node serve (Vercel, Render, Fly.io). Em Vercel/Render:
build `npm run build`, start `npm start`, e cadastre as mesmas variáveis do `.env.example`.
Defina `NEXT_PUBLIC_SITE_URL` com o endereço final (https).

## Dados da loja

Instagram, telefone, textos de medidas e frete ficam em `lib/store.ts`.

## O que ainda não tem

- E-mail automático de confirmação/rastreio (dá para ligar o Resend ou similar no `syncPayment` em `lib/orders.ts`).
- Reserva de estoque enquanto o Pix está aberto: o estoque baixa quando o pagamento é aprovado.
- Cupom de desconto, boleto, login de cliente e emissão de nota fiscal.
- Páginas de política de troca, privacidade e termos (necessárias para operar; crie antes de abrir).
