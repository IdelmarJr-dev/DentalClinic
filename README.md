<div align="center">

# 🦷 DentalClinic — Sistema de Gestão para Clínicas Odontológicas

**Uma plataforma completa de gestão clínica: agenda, prontuários, financeiro, funcionários e lembretes automáticos via WhatsApp.**

[![Demo ao vivo](https://img.shields.io/badge/demo-ao%20vivo-success?style=for-the-badge)](https://front-end-clinical.vercel.app)
![React](https://img.shields.io/badge/React-18-61DAFB?style=flat-square&logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-18+-339933?style=flat-square&logo=node.js&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-Postgres-3ECF8E?style=flat-square&logo=supabase&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-6-646CFF?style=flat-square&logo=vite&logoColor=white)

**🔗 [Acesse a demonstração ao vivo](https://front-end-clinical.vercel.app)**

</div>

---

## 📌 Sobre o projeto

O **DentalClinic** é um sistema full-stack de gestão para clínicas odontológicas, desenvolvido como parte da disciplina de **[Extensão Curricular I](https://suap.ifpi.edu.br/edu/disciplina/78813/)** do IFPI.

O sistema cobre o fluxo completo de uma clínica: cadastro e busca de pacientes, agendamento de consultas, controle financeiro, prontuários clínicos, gestão de acesso de funcionários por papel (admin, recepção) e um dashboard com métricas de faturamento. Um diferencial do projeto é a automação de **lembretes de consulta via WhatsApp**, disparados por um scheduler que roda em segundo plano no backend.

O projeto é dividido em dois repositórios:

- 🖥️ **ClinicRoot** — Frontend (SPA em React + TypeScript)
- ⚙️ **DentalClinic Backend** — API REST (Node.js + Express + Supabase)

---

## 🎥 Demonstração

> Substitua esta seção pelos prints/GIF reais do sistema em funcionamento.

<div align="center">

| Login | Dashboard |
|---|---|
| _`![login](./assets/login.png)`_ | _`![dashboard](./assets/dashboard.png)`_ |

| Agenda | Prontuários |
|---|---|
| _`![agenda](./assets/agenda.png)`_ | _`![prontuarios](./assets/prontuarios.png)`_ |

**GIF do fluxo completo (login → agendamento → lembrete disparado):**

`![demo](./assets/demo.gif)`

</div>

**🔗 Experimente você mesmo:** [front-end-clinical.vercel.app](https://front-end-clinical.vercel.app)

**Contas de demonstração:**

| Perfil | E-mail | Senha |
|---|---|---|
| Administrador | `admin@clinica.com` | `123456` |
| Recepção | `recepcao@clinica.com` | `123456` |


---

## 🏗️ Arquitetura do sistema

O sistema segue uma arquitetura desacoplada **SPA + API REST**, com autenticação própria via JWT (sem depender do Supabase Auth) e persistência em Postgres gerenciado pelo Supabase.

```
┌─────────────────────┐        HTTPS / REST        ┌──────────────────────────┐
│   ClinicRoot (SPA)   │ ──────────────────────────▶ │   DentalClinic Backend   │
│  React + TypeScript  │ ◀────────────────────────── │   Node.js + Express      │
│  Vite • Vercel        │      JWT Bearer Token       │   Deploy: Render         │
└─────────────────────┘                             └────────────┬─────────────┘
                                                                   │
                                                       ┌───────────▼────────────┐
                                                       │   Supabase (Postgres)   │
                                                       │  service_role key       │
                                                       └───────────┬────────────┘
                                                                   │
                                              ┌────────────────────▼───────────────────┐
                                              │  node-cron: fila de lembretes           │
                                              │  Integração com provider de WhatsApp    │
                                              └──────────────────────────────────────────┘
```

**Fluxo de autenticação:** o login é feito por e-mail/senha (hash **PBKDF2**), o backend emite um **JWT** assinado, e o frontend anexa esse token em todas as requisições protegidas via header `Authorization: Bearer <token>`.

**Fluxo de lembretes automáticos:** um job agendado (`node-cron`) varre periodicamente a agenda em busca de consultas próximas (janela configurável em horas), enfileira os lembretes pendentes e dispara as mensagens através da camada de integração com WhatsApp.

---

## 🛠️ Tecnologias utilizadas

### Frontend — ClinicRoot
- **React 18** + **TypeScript** — base da SPA
- **Vite 6** — build e dev server
- **React Router 7** — roteamento client-side, com proteção de rotas por papel de usuário
- **Tailwind CSS** — estilização utilitária
- **Radix UI** + **lucide-react** — componentes acessíveis e ícones
- **Recharts** — gráficos do dashboard de faturamento
- **Sonner** — notificações toast
- **jsPDF** + **html2canvas** — exportação de relatórios em PDF direto no navegador
- Deploy contínuo na **Vercel**

### Backend — DentalClinic API
- **Node.js** + **Express** + **TypeScript**
- **Supabase** (`@supabase/supabase-js`) como camada de persistência sobre Postgres, usando `service_role key`
- **JWT** (`jsonwebtoken`) para autenticação própria, desacoplada do Supabase Auth
- **PBKDF2** (`node:crypto`) para hash seguro de senhas
- **node-cron** para o agendamento e disparo automático de lembretes
- Deploy contínuo no **Render**

---

## 🚧 Desafios técnicos resolvidos

- **Autenticação própria desacoplada do provedor de dados**: em vez de usar o Supabase Auth, foi implementado um fluxo de autenticação independente (JWT + PBKDF2), permitindo controle total sobre papéis de usuário (admin, recepção, dentista) e regras de autorização por rota.
- **Automação confiável de lembretes**: construção de um serviço de fila (`services/reminders.ts`) que verifica periodicamente consultas próximas e evita disparos duplicados, integrado a um provider de WhatsApp plugável (com modo `mock` para desenvolvimento).
- **Cold start em ambiente free tier**: como o backend roda em um plano gratuito no Render (que "dorme" após inatividade), foi necessário lidar com o impacto disso na confiabilidade do scheduler de lembretes — mitigado com estratégias de keep-alive.
- **Sincronia de variáveis de ambiente entre build e runtime**: por se tratar de uma SPA com Vite, variáveis como a URL da API são embutidas no bundle *no momento do build* — isso exigiu disciplina de redeploy sempre que a configuração de ambiente muda entre local, staging e produção.
- **CORS estrito entre domínios diferentes**: frontend (Vercel) e backend (Render) rodam em domínios distintos, exigindo configuração precisa de origem permitida para evitar tanto falhas de CORS quanto brechas de segurança.
- **Proteção de rotas por papel no frontend e no backend**: as regras de acesso (ex.: apenas admin gerencia funcionários) são validadas em duas camadas — na interface (React Router) e na API (middleware `requireAuth` / `requireAdmin`) — evitando que a restrição dependa apenas da UI.

---

## 🔒 Sobre o código-fonte

O código deste projeto é privado por motivos de **propriedade intelectual/segurança**, mas sinta-se à vontade para explorar a demonstração!

**🔗 [front-end-clinical.vercel.app](https://front-end-clinical.vercel.app)**

---

## 🎓 Contexto acadêmico

Projeto desenvolvido como pré-requisito da disciplina **[Extensão Curricular I](https://suap.ifpi.edu.br/edu/disciplina/78813/)** — IFPI.
