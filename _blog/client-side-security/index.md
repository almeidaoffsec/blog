---
layout: post
title: "Não existe segurança no Client Side"
permalink: /client-side-security/
date: 2026-08-11
image: /client-side-security/client-side-auth-bypass.png
category: "segurança"
tags: ["Web", "Burp Suite", "Azure AD", "OWASP"]
description: "Como a interceptação de uma resposta JSON expôs toda a estrutura administrativa de uma aplicação corporativa e por que controle de acesso no frontend é ilusão."
---

Há algumas semanas, durante um teste de segurança em uma aplicação web corporativa, me deparei com um cenário que ilustra perfeitamente porque um desenvolvedor nunca deve confiar em validação de segurança no client side.

A aplicação em questão utilizava autenticação federada via Azure AD (Entra ID) um padrão robusto e bastante utilizado para eliminar a complexidade da criação de uma estrutura de autenticação própria. Após o login, o frontend consultava um endpoint que retornava os grupos organizacionais do usuário logado. Com base nessa resposta, a interface decidia quais módulos exibir: comercial, financeiro, administrativo, etc.

Até aqui o cenário é ideal, exceto pelo fato de que um usuário logado poderia interceptar a resposta do servidor e alterar as informações que o navegador está esperando.

## O teste óbvio

Durante a análise, usando o Burp Suite, interceptei a resposta de um endpoint que retornava metadados do usuário. Dentro do JSON havia um campo `groups`. Fiz o teste óbvio: alterei o valor do grupo para um de privilégio mais alto e reenviei a resposta ao navegador.

Voilà a aplicação renderizou toda a estrutura administrativa.

![Interceptação da resposta JSON no Burp Suite](./client-side-auth-bypass.png)

Botões, menus, formulários e chamadas que deveriam estar invisíveis para um usuário comum apareceram na tela como se eu tivesse permissão administrativa. Não precisei de exploit complexo, não precisei de engenharia social. Apenas troquei uma string em um JSON interceptado.

E o backend? Esse estava fazendo a parte dele apesar do acesso a toda a estrutura do frontend, qualquer chamada que tentasse manipular algum dado estava protegida por validação via Bearer Token.

## O verdadeiro impacto

Quando você expõe a estrutura interna de uma aplicação endpoints, parâmetros, fluxos de negócio, campos de formulário administrativos você não está apenas "mostrando botões que não funcionam". Você está:

- **Aumentando a superfície de ataque** do atacante, que agora sabe exatamente onde mirar
- **Revelando a lógica de negócio interna**, muitas vezes mais valiosa que os dados em si
- **Facilitando reconhecimento** para ataques direcionados como IDOR, mass assignment, etc.

## A causa raiz

O erro aqui é clássico: controle de acesso implementado no client side.

O frontend tomava decisões de segurança baseadas em dados que ele mesmo recebia e que podiam ser manipulados. É o equivalente a um segurança de banco deixar você entrar na área restrita porque você *disse* fazer parte do quadro de funcionários, ao invés de conferir crachá, digital ou qualquer credencial real.

A resposta do servidor deveria ser usada apenas para exibição de perfil nome, e-mail, avatar. **Autorização deve derivar de claims validadas no token JWT ou de verificações explícitas no backend a cada request.**

## Lições e recomendações

Se você trabalha com desenvolvimento ou segurança de aplicações, aqui vão algumas verificações rápidas:

- **Nunca use dados de `/users/me`** (ou similar) para autorização. Esse endpoint é para exibição, não para segurança.
- **Valide permissões no backend em toda chamada sensível.** O frontend é território inimigo.
- **Prefira Server-Side Rendering (SSR) ou Server Components** para decidir o que renderizar com base em permissões reais.
- **Utilize claims do token de identidade** (Azure AD, Okta, Auth0) para RBAC. Tokens assinados não podem ser alterados pelo cliente.

---

Essa descoberta foi reportada ao time de desenvolvimento com documentação técnica detalhada e já foi mitigada. Mas ela serve como um excelente lembrete:

> Segurança no frontend é ilusão de segurança.
