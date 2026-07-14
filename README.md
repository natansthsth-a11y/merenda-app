# Sis Lanchinho - v3.0

Projeto simples em HTML/JS puro (sem build) para montar pedidos de merenda em
tempo real, com área de admin, cadastro de produtos e controle de contas a pagar.
Pode ser hospedado de graça no GitHub Pages.

## 1. Criar o banco de dados (Supabase, gratuito)

1. Crie uma conta em https://supabase.com e um novo projeto (plano Free).
2. No painel do projeto, vá em **SQL Editor** e rode:

   ```sql
   -- Se já existirem tabelas antigas, apague
   drop table if exists merenda_itens cascade;
   drop table if exists pedidos cascade;
   drop table if exists contas_pagar cascade;
   drop table if exists produtos cascade;

   -- Tabela de pedidos
   create table pedidos (
     id bigint generated always as identity primary key,
     status text not null default 'aberto',
     valor_total numeric default 0,
     criado_em timestamptz not null default now(),
     fechado_em timestamptz
   );

   -- Tabela de itens do pedido
   create table merenda_itens (
     id bigint generated always as identity primary key,
     pedido_id bigint references pedidos(id) on delete cascade,
     nome_pessoa text not null,
     item text not null,
     quantidade int not null default 1,
     preco numeric default 0,
     criado_em timestamptz not null default now()
   );

   -- Tabela de contas a pagar
   create table contas_pagar (
     id bigint generated always as identity primary key,
     pedido_id bigint references pedidos(id) on delete cascade,
     nome_pessoa text not null,
     valor numeric not null default 0,
     pago boolean not null default false,
     criado_em timestamptz not null default now(),
     pago_em timestamptz
   );

   -- Tabela de produtos
   create table produtos (
     id bigint generated always as identity primary key,
     nome text not null,
     preco numeric default 0,
     criado_em timestamptz not null default now()
   );

   -- Habilitar RLS
   alter table pedidos enable row level security;
   alter table merenda_itens enable row level security;
   alter table contas_pagar enable row level security;
   alter table produtos enable row level security;

   -- Policies para pedidos
   create policy "Pedidos: inserir" on pedidos for insert with check (true);
   create policy "Pedidos: ler" on pedidos for select using (true);
   create policy "Pedidos: atualizar" on pedidos for update using (true);
   create policy "Pedidos: deletar" on pedidos for delete using (true);

   -- Policies para merenda_itens
   create policy "Itens: inserir" on merenda_itens for insert with check (true);
   create policy "Itens: ler" on merenda_itens for select using (true);
   create policy "Itens: atualizar" on merenda_itens for update using (true);
   create policy "Itens: deletar" on merenda_itens for delete using (true);

   -- Policies para contas_pagar
   create policy "Contas: inserir" on contas_pagar for insert with check (true);
   create policy "Contas: ler" on contas_pagar for select using (true);
   create policy "Contas: atualizar" on contas_pagar for update using (true);
   create policy "Contas: deletar" on contas_pagar for delete using (true);

   -- Policies para produtos
   create policy "Produtos: inserir" on produtos for insert with check (true);
   create policy "Produtos: ler" on produtos for select using (true);
   create policy "Produtos: atualizar" on produtos for update using (true);
   create policy "Produtos: deletar" on produtos for delete using (true);
   ```

3. Ative o **Realtime** nas quatro tabelas:

   ```sql
   alter publication supabase_realtime add table pedidos;
   alter publication supabase_realtime add table merenda_itens;
   alter publication supabase_realtime add table contas_pagar;
   alter publication supabase_realtime add table produtos;
   ```

4. Vá em **Settings > API** e copie a **Project URL** e a chave **anon public**.

## 2. Configurar o projeto

1. Copie `config.example.js` para `config.js` e preencha:

   ```js
   window.SUPABASE_URL = "https://SEU-PROJETO.supabase.co";
   window.SUPABASE_ANON_KEY = "SUA_CHAVE_ANON_PUBLICA";
   window.ADMIN_PASSWORD = "SUA_SENHA_ADMIN";
   ```

2. Defina uma senha segura para o admin no `ADMIN_PASSWORD`.

3. Teste localmente: abra `index.html` no navegador ou sirva a pasta com
   qualquer servidor estático.

4. Na primeira vez, cadastre os produtos pelo painel do admin (botão "Produtos").

## 3. Publicar no GitHub Pages

```bash
git init
git add index.html config.js config.example.js README.md
git commit -m "Sis Lanchinho v3.0"
git branch -M main
git remote add origin https://github.com/SEU-USUARIO/SEU-REPO.git
git push -u origin main
```

Depois, no GitHub: **Settings > Pages > Source: branch `main`, pasta `/`**.

## Como funciona

### Login
- Cada pessoa digita o seu nome para entrar.
- Se quiser acessar como admin, clica em "Sou admin" e digita a senha.

### Usuário
- Se tem pedido aberto, pode adicionar itens à lista.
- O autocompletar mostra produtos cadastrados pelo admin (com preço).
- Também pode digitar itens que não estão cadastrados e definir o valor.
- Se tem dívida pendente, aparece o valor total e quantas contas estão abertas.
- Só pode remover itens que ele mesmo adicionou.

### Admin
- **Abrir Pedido**: cria um novo pedido para as pessoas adicionarem itens.
- **Fechar Pedido**: informa o valor total, o sistema divide igualmente entre os participantes e gera as contas a pagar.
- **Produtos**: cadastra, edita e exclui produtos disponíveis. Aparece no autocompletar de todos.
- **Contas a Pagar**: visualiza todas as contas, pode alterar valores, dar baixa (marcar como pago) ou excluir.
- **Limpar Banco**: apaga todos os dados do sistema (pedidos, itens, contas e produtos).

### Real-time
- Todas as alterações aparecem instantaneamente para todos os usuários com a página aberta (via Supabase Realtime).
