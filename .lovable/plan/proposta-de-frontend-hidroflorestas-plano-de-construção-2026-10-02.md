# Proposta de frontend HidroFlorestas — plano de construção

Esta é uma demonstração navegável em pt-BR. Todos os dados são sintéticos e passam por um adapter substituível. Não terá backend, banco, Lovable Cloud, autenticação real nem cálculo IHFR. A especificação enviada é a fonte de verdade. As imagens servem só de referência de composição.

## Identidade visual
- Fundo `#F9FAFB`, superfícies brancas e campos em `#EFEFEF`. Texto em `#3E3E3E`.
- Acentos controlados: verde (green-600/700, `#00B51A`), azul `#0084DD` e ocre (amber-700, `#A1640B`). Erros em red-700.
- Fonte Poppins. Cartões com raio de 20px, controles com 10px e sombra suave `0 4px 18px -3px rgba(0,0,0,.25)`.
- Marca provisória em texto ("HIDRO" em azul e "FLORESTAS" em verde), sem logotipo novo.
- Ícones Lucide sempre acompanhados de rótulo. Sem neon, brilho ou gradientes fortes.
- O contraste será medido. Qualquer ajuste de cor fica registrado no README.

## Estrutura de navegação
- **Desktop:** cabeçalho branco com a marca, a pessoa e o laboratório atual, mais navegação lateral compacta com rótulos (Resumo, Áreas, Mapa e Membros, este último só para o proprietário). O botão Sair em ocre fica sempre visível.
- **Celular:** barra inferior com texto e Sair acessível.
- **Telas profundas:** trilha de contexto e botão voltar.
- **Painel "Demonstração":** flutuante, separado do produto e fácil de remover. Permite escolher o cenário:
  - perfis: OWNER, ADMIN contextual, MEMBER, ADMIN global, conta sem laboratório e laboratório inativo;
  - falhas: atraso, falha de rede, localização negada ou tardia, mapa indisponível, falha ao sair e conflitos;
  - variações de IHFR: ausente, vigente, insuficiente, incompatível e resultado desconhecido.

## Telas (todas da tabela da proposta)
1. `/`: apresentação curta com Entrar e Criar conta.
2. `/login`: cartão de cerca de 500px, mostrar/ocultar senha e mensagem genérica de erro. ADMIN global vai para `/admin`.
3. `/register`: painel ilustrado à esquerda e formulário à direita, nos campos exatos da proposta.
4. `/logout`: estados de andamento, erro com tentar novamente e sucesso.
5. `/workspace`:
   - acolhimento como na referência 4;
   - lista de laboratórios e criação de laboratório;
   - limite de 5 laboratórios;
   - "Entrar em laboratório" marcado como "Ainda indisponível";
   - diálogo de configurações com desativar/excluir, exigindo digitar o nome exato.
6. Resumo do laboratório: totais, histórico paginado com "Carregar mais" e links para áreas e coletas.
7. Áreas: grade de cartões com ícone neutro (sem foto), nome, município/UF, coordenadas e "Ver detalhes". "Nova área" aparece só quando permitido.
8. Nova área:
   - nome, latitude/longitude e campos opcionais;
   - ponto escolhido por clique no mapa ou pela localização do dispositivo, só após ação explícita;
   - revisão antes de cadastrar.
9. Detalhe da área: mapa com o ponto, informações, data de cadastro e "Registrar coleta".
10. Nova coleta: ocorrência inicializada uma única vez, fuso do dispositivo ou offset manual, validações e revisão (corrigir ou confirmar). Mostra o RFC3339 equivalente.
11. Detalhe da coleta:
    - ocorrência e confirmação exibidas separadamente;
    - link para os dados ambientais;
    - seção "Diagnóstico IHFR experimental" com aviso científico, componentes W/S/V/T e detalhes técnicos recolhíveis;
    - criar, substituir e revogar com confirmação, além da recuperação de tentativa incerta.
12. Dados ambientais: consulta do conjunto confirmado ou da ausência.
13. Novo conjunto ambiental:
    - quatro grupos em acordeão colorido (Água em azul, Solo em ocre, Vegetação em verde, Terreno em neutro);
    - campos e domínios exatos da proposta, com decimais em pt-BR;
    - revisão antes de confirmar;
    - tratamento de null, zero e falso;
    - regra de profundidade do poço.
14. Mapa territorial: Leaflet com tiles OSM e atribuição visível, lista alternativa por teclado (incluindo áreas sem localização), painel da área selecionada e coletas confirmadas. Se o mapa falhar, a lista continua utilizável.
15. Membros: o proprietário promove ou rebaixa entre MEMBER e ADMIN.
16. `/admin` e `/admin/users`:
    - busca, filtros e paginação;
    - detalhe da conta;
    - mudança de papel ou estado com justificativa, revisão antes/depois e tratamento de conflito;
    - auditoria;
    - em telas pequenas, a tabela vira cartões.
17. Redirecionamentos de `/dashboard`, `/dashboard/collects` e `/dashboard/admin/users`.

Toda tela pertinente mostra estes estados: carregando (sem números zerados), vazio, falha com repetir, somente leitura, sem permissão e "recurso não encontrado" genérico.

## Acessibilidade e responsividade
- Funciona em 375, 768 e 1440px e com zoom de 200%.
- Áreas de toque de 44px e foco visível.
- Avisos para leitores de tela (`role=status` e `role=alert`), erros ligados a cada campo e foco no primeiro campo inválido.
- Diálogos acessíveis.
- Status sempre com texto, nunca só com cor.
- Respeita a preferência por movimento reduzido.

## Detalhes técnicos
- Tokens em `src/styles.css` (oklch). Poppins carregada por `<link>` no root.
- `src/domain/`: tipos dos DTOs com nomes e enums exatos, constantes de versão IHFR e rótulos pt-BR.
- `src/adapter/`:
  - uma interface `HidroApi` e uma implementação mock em memória com fixtures, atrasos e falhas controlados pelo cenário;
  - Idempotency-Key estável por tentativa;
  - envelopes distintos por domínio;
  - resultados IHFR como fixtures rotuladas "Resultado simulado — demonstração".
- `src/demo/`: armazenamento de cenário (React context) e o painel.
- `src/features/<domínio>/`: componentes de apresentação e estado de formulário separados do acesso a dados.
- Rotas TanStack com prefixo `/dashboard/laboratories/$laboratoryId/...`. Os guardas de papel ficam num módulo substituível.
- Leaflet e react-leaflet carregados só no cliente (lazy + ClientOnly). Utilitários de data e offset com testes vitest: −03:00, Z, +05:45 e limites.
- O README documenta como executar, a estrutura, os cenários e os estados, e as adaptações necessárias para Next.js App Router. Nada de migração automática.
- Metadados `head()` próprios em cada rota.
