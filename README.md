# Modelagem e Normalização do Banco de Dados de uma Escola

*Atividade acadêmica — Modelagem de Dados / Banco de Dados*

---

## 1. Cenário da escola

Pensa assim: você foi chamado pra organizar a "bagunça de papel" de uma escola e transformar isso num banco de dados. O que essa escola precisa guardar, na prática?

- Dados dos **alunos** (quem são, quando nasceram, como contatar).
- Dados dos **professores** (quem são, especialidade, contato).
- As **disciplinas** que existem no currículo (Matemática, Português, etc.), com carga horária.
- As **turmas** (ex: "9º Ano A – 2026 – Manhã").
- Qual **professor leciona qual disciplina em qual turma** (o "grade de aulas").
- A **matrícula** de cada aluno numa turma (o vínculo aluno↔turma↔ano).
- As **notas** de cada aluno em cada disciplina.

O enunciado sugere alunos, professores, disciplinas, turmas, matrículas e notas como candidatos a entidade. Mas nem tudo que parece "coisa" é entidade — **Matrícula** e **Nota**, por exemplo, não são "objetos do mundo" como Aluno ou Professor: elas nascem de um *relacionamento* (aluno se relaciona com turma → isso é a matrícula; matrícula se relaciona com disciplina → isso é a nota). São o que chamamos de **entidades associativas**, e isso vai ficar claro naturalmente durante a normalização — não precisamos decidir isso de cabeça agora, o processo de normalização mesmo vai "revelar" essa estrutura.

---

## 2. Entidade inicial e atributos (Forma Não Normalizada — 0FN)

Imagina que, antes de qualquer modelagem, a secretaria da escola só usa **uma ficha só**, tipo um formulário de papel, pra registrar a matrícula de um aluno com todas as disciplinas que ele cursa. Uma única "tabela-mãe":

**FICHA_MATRICULA (0FN — não normalizada)**

| id_aluno | nome_aluno | data_nascimento_aluno | email_aluno | id_turma | nome_turma | ano_letivo | turno | disciplinas (grupo repetitivo) |
|---|---|---|---|---|---|---|---|---|
| 1 | Ana Souza | 12/03/2011 | ana@escola.com | 10 | 9ºA | 2026 | Manhã | **[** {id_disciplina: 1, nome_disciplina: Matemática, carga_horaria: 80, id_professor: 5, nome_professor: Carlos, email_professor: carlos@escola.com, nota1: 8, nota2: 7}, {id_disciplina: 2, nome_disciplina: Português, carga_horaria: 80, id_professor: 6, nome_professor: Marta, email_professor: marta@escola.com, nota1: 9, nota2: 9} **]** |

Repare que dentro da célula "disciplinas" existe **uma lista inteira** — um grupo repetitivo. Isso é o retrato clássico da forma não normalizada.

**Todos os atributos dessa estrutura:**
id_aluno, nome_aluno, data_nascimento_aluno, email_aluno, id_turma, nome_turma, ano_letivo, turno, id_disciplina, nome_disciplina, carga_horaria, id_professor, nome_professor, email_professor, nota1, nota2.

**Chave primária candidata:** id_aluno + id_turma (mas ela não identifica cada linha de forma única, porque cada aluno tem várias disciplinas dentro da mesma ficha — é justamente esse o problema que a 1FN resolve).

**Problemas de redundância evidentes:**
- O nome e e-mail do professor se repetem toda vez que ele aparece em disciplinas de turmas diferentes.
- Os dados da turma (nome_turma, ano_letivo, turno) se repetem para cada aluno da mesma turma.
- Os dados do aluno se repetiriam de novo se ele tivesse mais de uma ficha (ex: em anos diferentes).

---

## 3. Problemas e anomalias

Usando esse cenário de "ficha única", dá pra ver claramente os três clássicos problemas:

**Anomalia de inserção**
Não dá pra cadastrar um professor novo, ou uma disciplina nova no currículo, **sem que exista um aluno matriculado**. Se a escola contrata a professora Marta pra lecionar Geografia mas ainda não tem nenhum aluno matriculado nessa disciplina, não existe "lugar" pra guardar essa informação — porque professor e disciplina só existem dentro da ficha de um aluno.

**Anomalia de alteração (atualização)**
Se o e-mail do professor Carlos mudar, é preciso alterar **em todas as fichas de todos os alunos** que ele leciona. Esquecer de atualizar uma única linha gera inconsistência (um sistema "acha" que ele tem dois e-mails diferentes).

**Anomalia de exclusão**
Se a Ana for a única aluna matriculada em Matemática numa turma e ela for desmatriculada, ao apagar a ficha dela **perde-se também a informação de que a disciplina Matemática existe, sua carga horária, e que o professor Carlos a leciona** — mesmo que esses dados continuem sendo verdadeiros e relevantes pra escola.

---

## 4. Primeira Forma Normal (1FN)

**Regra da 1FN:** cada célula da tabela deve conter um único valor atômico (nada de listas dentro de célula), e não pode haver grupos repetitivos dentro de uma mesma linha.

O grupo repetitivo aqui é o bloco "disciplinas" (que guardava uma lista de disciplinas por aluno). A solução é **"achatar"** a tabela: cada combinação aluno+turma+disciplina vira uma linha própria.

**Antes (0FN):** 1 linha por aluno, com uma lista de disciplinas dentro.
**Depois (1FN):** 1 linha por combinação (aluno, turma, disciplina).

**MATRICULA_DISCIPLINA (1FN)**

| id_aluno | nome_aluno | data_nasc. | email_aluno | id_turma | nome_turma | ano_letivo | turno | id_disciplina | nome_disciplina | carga_horaria | id_professor | nome_professor | email_professor | nota1 | nota2 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | Ana Souza | 12/03/2011 | ana@escola.com | 10 | 9ºA | 2026 | Manhã | 1 | Matemática | 80 | 5 | Carlos | carlos@escola.com | 8 | 7 |
| 1 | Ana Souza | 12/03/2011 | ana@escola.com | 10 | 9ºA | 2026 | Manhã | 2 | Português | 80 | 6 | Marta | marta@escola.com | 9 | 9 |

Nenhuma célula tem mais de um valor agora — atomicidade garantida. **Chave primária:** como uma linha só existe uma vez pra cada combinação de aluno+turma+disciplina, a PK vira composta: **(id_aluno, id_turma, id_disciplina)**.

Só que repara: essa tabela ainda está *cheia* de redundância — o nome da Ana se repete em cada disciplina dela, os dados da turma se repetem, os dados do professor se repetem. A 1FN resolveu o problema de atomicidade, mas não o de redundância. É pra isso que existem a 2FN e a 3FN.

---

## 5. Segunda Forma Normal (2FN)

**Regra da 2FN:** a tabela já deve estar na 1FN, e **todo atributo não-chave precisa depender da chave inteira** — nenhum atributo pode depender de apenas uma *parte* de uma chave composta (isso só é um problema relevante quando a chave primária é composta, que é exatamente o nosso caso: id_aluno + id_turma + id_disciplina).

Vamos checar, atributo por atributo, de que parte da chave ele realmente depende:

| Atributo | Depende de | Tipo de dependência |
|---|---|---|
| nome_aluno, data_nascimento, email_aluno | id_aluno (só) | **Parcial** |
| nome_turma, ano_letivo, turno | id_turma (só) | **Parcial** |
| nome_disciplina, carga_horaria | id_disciplina (só) | **Parcial** |
| id_professor, nome_professor, email_professor | id_turma + id_disciplina (não precisa do aluno!) | **Parcial** |
| nota1, nota2 | id_aluno + id_turma + id_disciplina (os três juntos) | **Total (ok)** |

Ou seja: quase tudo dependia só de um pedaço da chave. Isso é o que causava a redundância. Vamos separar cada grupo numa entidade própria:

- **ALUNO** (o que depende só de id_aluno)
- **TURMA** (o que depende só de id_turma)
- **DISCIPLINA** (o que depende só de id_disciplina)
- **TURMA_DISCIPLINA** (o que depende de id_turma + id_disciplina — quem leciona o quê, em qual turma)
- **NOTA** (o que realmente precisa dos três: aluno, turma e disciplina)

**Resultado após a 2FN:**

- `ALUNO(id_aluno PK, nome, data_nascimento, email)`
- `TURMA(id_turma PK, nome_turma, ano_letivo, turno)`
- `DISCIPLINA(id_disciplina PK, nome_disciplina, carga_horaria)`
- `TURMA_DISCIPLINA(id_turma FK, id_disciplina FK, id_professor, nome_professor, email_professor)` — PK: (id_turma, id_disciplina)
- `NOTA(id_aluno FK, id_turma FK, id_disciplina FK, nota1, nota2)` — PK: (id_aluno, id_turma, id_disciplina)

Já eliminamos boa parte da redundância. Mas ainda sobrou um resto de problema dentro de `TURMA_DISCIPLINA`: nome_professor e email_professor não descrevem a *turma-disciplina* em si, descrevem o **professor**. É esse resíduo que a 3FN vai limpar.

---

## 6. Terceira Forma Normal (3FN)

**Regra da 3FN:** a tabela já deve estar na 2FN, e **nenhum atributo não-chave pode depender de outro atributo não-chave** (só pode depender da chave). Isso se chama **dependência transitiva**: A depende de B, e B depende da chave — só que A deveria depender diretamente da chave, e não está.

**Onde está a dependência transitiva?**

Em `TURMA_DISCIPLINA(id_turma, id_disciplina, id_professor, nome_professor, email_professor)`:
`id_turma + id_disciplina → id_professor → nome_professor, email_professor`

Ou seja: nome_professor e email_professor não dependem diretamente da chave (id_turma+id_disciplina) — eles dependem do id_professor, que é só mais um atributo comum ali dentro. Isso é dependência transitiva clássica. Solução: tirar os dados do professor pra uma entidade própria.

- **PROFESSOR**`(id_professor PK, nome, email, formacao)`
- `TURMA_DISCIPLINA` fica só com: `id_turma FK, id_disciplina FK, id_professor FK` — PK (id_turma, id_disciplina)

**E os campos calculados?**
O enunciado original (na ficha de papel) também poderia incluir uma "nota_final" (média entre nota1 e nota2) e uma "situação" (Aprovado/Reprovado). Se guardássemos isso como coluna, teríamos:

`nota1, nota2 → nota_final → situação`

— outra dependência transitiva na entidade NOTA, só que aqui o "atributo não-chave dependendo de outro não-chave" é literalmente uma fórmula (nota_final = média de nota1 e nota2; situação = "Aprovado" se nota_final ≥ 6). **Esses dois campos não devem ser armazenados**: eles são calculados a partir de nota1 e nota2 sempre que forem necessários (numa consulta SQL, numa view, ou na aplicação). Guardá-los fisicamente é redundância pura e cria risco de inconsistência (ex: alguém corrige nota1 e esquece de recalcular nota_final).

**Resultado final após a 3FN:** `NOTA(id_aluno FK, id_turma FK, id_disciplina FK, nota1, nota2)` — sem nota_final e sem situação armazenadas.

**Ajuste final de modelagem:** o par (id_aluno + id_turma) representa, na prática, a matrícula do aluno naquela turma — e essa matrícula tem vida própria (tem data, pode ser trancada, etc.), então faz sentido dar a ela uma identidade própria em vez de repetir sempre os dois IDs juntos. Criamos a entidade **MATRICULA** com uma chave própria (id_matricula), e a NOTA passa a se referir à matrícula + disciplina, em vez de aluno + turma + disciplina diretamente. Isso não muda nenhuma regra de normalização (é só uma simplificação estrutural comum na prática), mas deixa o modelo mais limpo e mais fácil de evoluir (por exemplo, adicionar "status da matrícula" no futuro).

---

## 7. Entidades finais

- **ALUNO**
- **PROFESSOR**
- **DISCIPLINA**
- **TURMA**
- **TURMA_DISCIPLINA** (associativa: quem leciona o quê, em qual turma)
- **MATRICULA** (associativa: vínculo aluno↔turma)
- **NOTA** (associativa: vínculo matrícula↔disciplina)

---

## 8. PKs e FKs — formato final das entidades

**ALUNO**
- PK id_aluno
- nome
- data_nascimento
- email

**PROFESSOR**
- PK id_professor
- nome
- email
- formacao

**DISCIPLINA**
- PK id_disciplina
- nome_disciplina
- carga_horaria

**TURMA**
- PK id_turma
- nome_turma
- ano_letivo
- turno

**TURMA_DISCIPLINA**
- PK id_turma
- PK FK id_turma
- PK FK id_disciplina
- FK id_professor

**MATRICULA**
- PK id_matricula
- FK id_aluno
- FK id_turma
- data_matricula

**NOTA**
- PK id_matricula
- PK FK id_matricula
- PK FK id_disciplina
- nota1
- nota2

*(Observação de formatação: nas duas entidades associativas — TURMA_DISCIPLINA e NOTA — a chave primária é composta pelas próprias chaves estrangeiras, por isso elas aparecem marcadas como "PK FK" ao mesmo tempo.)*

---

## 9. Relacionamentos e cardinalidades

- **PROFESSOR 1:N TURMA_DISCIPLINA** — um professor pode lecionar várias combinações turma/disciplina; cada combinação turma/disciplina tem um professor responsável.
- **DISCIPLINA 1:N TURMA_DISCIPLINA** — uma disciplina pode ser oferecida em várias turmas; cada linha de TURMA_DISCIPLINA se refere a uma única disciplina.
- **TURMA 1:N TURMA_DISCIPLINA** — uma turma tem várias disciplinas em seu currículo.
- ⇒ Na origem, **TURMA N:N DISCIPLINA** (uma turma tem várias disciplinas, e uma disciplina é dada em várias turmas) — resolvido pela associativa TURMA_DISCIPLINA.
- **ALUNO 1:N MATRICULA** — um aluno pode ter várias matrículas (por exemplo, em anos letivos diferentes).
- **TURMA 1:N MATRICULA** — uma turma possui vários alunos matriculados.
- ⇒ Na origem, **ALUNO N:N TURMA** (ao longo do tempo um aluno passa por várias turmas, e uma turma tem vários alunos) — resolvido pela associativa MATRICULA.
- **MATRICULA 1:N NOTA** — uma matrícula gera uma nota para cada disciplina cursada naquela turma.
- **DISCIPLINA 1:N NOTA** — uma disciplina acumula notas de várias matrículas diferentes.
- ⇒ Na origem, **MATRICULA N:N DISCIPLINA** — resolvido pela associativa NOTA.

---

## 10. Entidade associativa — por que ela é necessária

Um relacionamento **N:N** não pode ser representado diretamente no modelo relacional (uma tabela não comporta "muitos para muitos" sozinha, porque uma FK só guarda um valor por linha). A solução padrão é criar uma **entidade associativa**: uma tabela nova cujo objetivo é justamente representar aquele encontro entre as duas entidades, carregando as duas chaves estrangeiras (e, às vezes, atributos próprios daquele relacionamento).

No nosso modelo, isso aconteceu três vezes:
- **TURMA_DISCIPLINA** resolve TURMA×DISCIPLINA (e ainda carrega o atributo "quem leciona").
- **MATRICULA** resolve ALUNO×TURMA (e carrega o atributo "data_matricula").
- **NOTA** resolve MATRICULA×DISCIPLINA (e carrega os atributos "nota1, nota2").

Sem essas entidades, seria impossível, por exemplo, um aluno ter notas em várias disciplinas ao mesmo tempo sem duplicar todos os seus dados pessoais em cada linha — voltaríamos exatamente pro problema da ficha única do início.

---

## Modelo final do banco (visão geral tipo DER textual)

```
ALUNO (id_aluno PK) ──┐
                       │ 1:N
                       ▼
                  MATRICULA (id_matricula PK, id_aluno FK, id_turma FK)
                       ▲
                       │ 1:N
TURMA (id_turma PK) ───┘
   │
   │ 1:N
   ▼
TURMA_DISCIPLINA (id_turma PK/FK, id_disciplina PK/FK, id_professor FK)
   ▲                              ▲
   │ 1:N                         │ 1:N
DISCIPLINA (id_disciplina PK)    PROFESSOR (id_professor PK)

MATRICULA (id_matricula PK) ──┐
                                │ 1:N
                                ▼
                          NOTA (id_matricula PK/FK, id_disciplina PK/FK, nota1, nota2)
                                ▲
                                │ 1:N
DISCIPLINA (id_disciplina PK) ──┘
```

**Lista de entidades:** ALUNO, PROFESSOR, DISCIPLINA, TURMA, TURMA_DISCIPLINA, MATRICULA, NOTA
**PKs:** cada entidade "principal" tem PK própria (id_aluno, id_professor, id_disciplina, id_turma); as associativas têm PK composta pelas FKs que herdam (ou surrogate, como id_matricula).
**FKs:** TURMA_DISCIPLINA → TURMA, DISCIPLINA, PROFESSOR | MATRICULA → ALUNO, TURMA | NOTA → MATRICULA, DISCIPLINA.

---

## 11. Justificativa da normalização até a 3FN

- **1FN garantida:** todas as tabelas finais têm apenas valores atômicos em cada célula, e não existe nenhum grupo repetitivo — cada disciplina de um aluno virou uma linha própria em NOTA, em vez de uma lista dentro de uma célula.
- **2FN garantida:** todo atributo não-chave depende da **chave inteira** de sua entidade. Nome e e-mail do aluno dependem só de id_aluno (e estão em ALUNO, cuja chave é id_aluno — dependência total). O mesmo vale para TURMA, DISCIPLINA e PROFESSOR. Não sobrou nenhum atributo "preso" a apenas parte de uma chave composta.
- **3FN garantida:** não existe mais nenhum atributo não-chave dependendo de *outro* atributo não-chave. Os dados do professor saíram de dentro de TURMA_DISCIPLINA e viraram a entidade PROFESSOR — agora id_professor é FK, e nome/email do professor dependem diretamente da PK de PROFESSOR, não mais de um atributo vizinho. Os campos calculados (nota_final, situação) foram removidos do armazenamento físico, porque dependiam de outros atributos não-chave (nota1 e nota2) e podem ser obtidos por cálculo sempre que necessário.

**Redundâncias eliminadas:** nome/e-mail do aluno não se repete mais por disciplina; dados da turma não se repetem por aluno; dados do professor não se repetem por turma em que ele leciona; notas calculadas não ficam duplicadas e sujeitas a ficar desatualizadas.

**Anomalias eliminadas:**
- **Inserção:** agora dá pra cadastrar um professor, uma disciplina ou uma turma nova mesmo sem nenhum aluno matriculado ainda, porque cada uma é uma entidade independente.
- **Alteração:** o e-mail de um professor é alterado em um único lugar (a linha dele em PROFESSOR), e reflete automaticamente em todas as turmas que ele leciona, via FK.
- **Exclusão:** apagar a matrícula de um único aluno não apaga mais os dados da disciplina, do professor ou da turma, porque eles existem de forma independente.

---

## Revisão final ✅

- [x] Todas as entidades possuem chave primária (id_aluno, id_professor, id_disciplina, id_turma, e as compostas/surrogate das associativas).
- [x] As chaves estrangeiras estão corretas e apontam para PKs existentes (TURMA_DISCIPLINA → TURMA/DISCIPLINA/PROFESSOR; MATRICULA → ALUNO/TURMA; NOTA → MATRICULA/DISCIPLINA).
- [x] Não existem atributos repetitivos (o grupo "disciplinas" da ficha original foi eliminado na 1FN).
- [x] Não existem dependências parciais (todo atributo depende da chave inteira de sua tabela, resolvido na 2FN).
- [x] Não existem dependências transitivas (dados do professor isolados na 3FN; campos calculados removidos).
- [x] Os relacionamentos fazem sentido para o contexto de uma escola (aluno se matricula em turma, turma tem disciplinas com professores responsáveis, matrícula gera notas por disciplina).
