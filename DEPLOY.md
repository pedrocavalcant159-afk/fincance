# Publicação do Controle Financeiro no plano gratuito Spark

Esta versão usa Firebase Authentication, Cloud Firestore e Firebase Hosting. Ela não depende de Cloud Functions e pode ser publicada no plano gratuito Spark, respeitadas as cotas do Firebase.

## Antes de publicar

1. Substitua todos os campos entre colchetes em `termos.html` e `privacidade.html`:
   - nome empresarial ou nome completo;
   - CPF/CNPJ;
   - endereço;
   - e-mail de suporte;
   - e-mail de privacidade/encarregado;
   - provedor de pagamentos;
   - caminho de cancelamento e política comercial de reembolso.
2. Peça revisão dos documentos a um advogado especializado em contratos digitais, consumidor e LGPD.
3. Ative autenticação multifator na conta Google que administra o projeto Firebase.
4. Revise quem possui acesso ao projeto no Google Cloud IAM.
5. Ative o Firebase App Check antes da abertura pública.

## Preparar o Firebase

No Console do Firebase:

1. Abra **Authentication > Sign-in method** e habilite **E-mail/senha**.
2. Crie o banco em **Firestore Database**, preferencialmente em modo de produção.
3. Confirme que o aplicativo Web usa o projeto `financeiro-familia-44fac` ou substitua a configuração Firebase nos arquivos `index.html` e `admin.html`.

## Publicar regras e site

Instale a Firebase CLI e autentique-se:

```powershell
npm install -g firebase-tools
firebase login
firebase use financeiro-familia-44fac
firebase deploy --only firestore:rules,hosting
```

As regras em `firestore.rules` substituem as regras atualmente publicadas. Se o projeto já estiver em produção, teste-as primeiro no Firebase Emulator Suite.

## Criar o primeiro administrador sem servidor

1. Crie sua conta normalmente pelo cadastro do Controle Financeiro.
2. No Console do Firebase, abra **Authentication > Users** e copie o UID da sua conta.
3. Abra **Firestore Database > Data**.
4. Crie a coleção `admins`.
5. Dentro dela, crie um documento cujo ID seja exatamente o UID copiado.
6. Adicione estes campos:

| Campo | Tipo | Valor |
|---|---|---|
| `active` | boolean | `true` |
| `role` | string | `owner` |
| `email` | string | seu e-mail administrativo |

7. Abra `/admin.html` e entre com o mesmo e-mail e senha.

Não crie nenhuma função no site que permita gravar na coleção `admins`. Pelas regras fornecidas, somente o Console Firebase ou uma credencial administrativa externa pode criar, alterar ou excluir administradores.

## Como funciona o bloqueio no Spark

O painel grava `accountDisabled: true` no perfil do cliente. As regras do Firestore passam a negar acesso às transações, gastos fixos, eventos e notas, e o aplicativo desconecta a conta em tempo real.

Esse bloqueio protege o aplicativo e os dados, mas não remove nem desativa o registro interno do Firebase Authentication. Para desativar completamente a identidade, use manualmente **Authentication > Users > Disable account** no Console Firebase.

## Privacidade do painel master

O painel lê somente documentos da coleção `users`, que contêm:

- UID, nome e e-mail;
- conta ativa ou bloqueada;
- criação e último acesso;
- plano e situação da assinatura;
- versões dos documentos jurídicos aceitos.

As regras não concedem ao administrador acesso a `transactions`, `fixos`, `events`, `eventExpenses` ou `notes`. As ações administrativas criam registros em `adminAuditLogs`, coleção que não pode ser lida ou alterada pelo navegador.

O botão de redefinição pede ao Firebase Authentication que envie um link oficial ao cliente. O administrador não vê a senha atual nem a nova senha.

## Cobrança

O painel permite administrar plano e situação manualmente no Spark. Uma venda automática com Stripe, Mercado Pago ou outro provedor exige um webhook confiável fora do navegador. Essa integração poderá exigir infraestrutura paga; nunca libere um plano apenas com uma alteração enviada pelo próprio cliente.
