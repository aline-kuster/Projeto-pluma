# 📄 Relatório Integrado de Análise e Justificativa da Segmentação com SAM 2

## Objetivo do Projeto
O objetivo principal desta atividade consistiu em desenvolver, avaliar e justificar melhorias na segmentação de uma pluma de fumaça gerada pelo incêndio no **Parque Estadual do Juquery (Franco da Rocha/SP)**. O trabalho buscou superar as limitações da segmentação 100% automática, que frequentemente comete erros de *supersegmentação* ao englobar indevidamente partes do céu, nuvens, edifícios ou o terreno do entorno.

Para viabilizar a extração precisa da Região de Interesse (ROI), a atividade focou em **produzir uma máscara binária** — uma matriz digital do mesmo tamanho da imagem original que atribui o valor `1` (cor branca) para os pixels pertencentes à fumaça e o valor `0` (cor preta) para o fundo. A criação dessa máscara é indispensável para fins de modelagem ambiental, pois possibilita:
* Isolar a fumaça do cenário;
* Quantificar a sua área em pixels;
* Determinar a localização do seu centroide (centro de massa);
* Acompanhar a sua trajetória de dispersão transportada pelo vento.

---

## Metodologia e Configurações de Prompts
A metodologia empregada envolveu o teste de **três configurações diferentes de prompts** no modelo **SAM 2**, apresentadas obrigatoriamente no formato de três painéis lado a lado para validação visual direta:
* **Painel 1:** Imagem original sobreposta pelos prompts fornecidos ao modelo (*bounding box* e/ou pontos positivos verdes e negativos vermelhos);
* **Painel 2:** Máscara binária resultante;
* **Painel 3:** Máscara sobreposta em tom vermelho translúcido sobre a foto original para identificação visual de falhas de ajuste.

### Configurações Avaliadas:
1. **Configuração 1:** Utilização de apenas uma *bounding box* restrita ao tronco da pluma.
2. **Configuração 2:** Utilização de múltiplos pontos positivos soltos na fumaça e pontos negativos de descarte no fundo.
3. **Configuração 3:** Abordagem híbrida utilizando uma *bounding box* ampla refinada por pontos negativos.

---

## Avaliação Qualitativa do Ajuste das Máscaras

A avaliação detalhada das máscaras geradas pelo código revelou comportamentos distintos ao longo do corpo da pluma:

* **Base da Pluma (Excelente Definição):** Em todas as três configurações testadas, a base da fumaça ficou muito bem definida. Esse alto nível de precisão ocorreu devido ao forte contraste cromático e de brilho existente entre a fumaça inicial e o solo escuro do terreno, gerando um gradiente de borda nítido que o SAM 2 identificou facilmente.
* **Fumaça Apagada/Difusa na Direita (Subsegmentação):** Nenhuma das três configurações conseguiu capturar a fumaça mais apagada situada no lado direito. Por ser uma fumaça rala e semitransparente, a luz do céu claro atravessa o gás, fazendo com que a tonalidade dos pixels dessa região se confunda com a cor do próprio céu. Como o modelo depende de linhas de contorno marcantes, essa fumaça difusa foi interpretada como fundo e ficou totalmente de fora.
* **Franjas no Lado Esquerdo (Subsegmentação Parcial):** Partes da fumaça no lado esquerdo da pluma também ficaram de fora da segmentação em todas as abordagens. Por apresentarem menor densidade e tonalidade mais esbranquiçada em comparação ao núcleo escuro do tronco central, o modelo priorizou apenas a massa mais espessa, omitindo as franjas vaporosas da esquerda na ausência de prompts dedicados.
* **Controle de Supersegmentação:** A inclusão indevida de regiões de fundo (céu, nuvens e morros) foi contida com sucesso, principalmente nas Configurações 2 e 3, devido ao uso estratégico dos **pontos negativos**, que ancoraram os limites da máscara e impediram o vazamento para o fundo de baixo contraste.

---

## Desafios Técnicos e Características da Imagem
Diversas características inerentes à imagem dificultaram a segmentação computacional:
1. **Natureza Fluida e Não-Rígida:** Ausência de fronteiras físicas delimitadas e contornos rígidos.
2. **Heterogeneidade Cromática na Base:** Presença simultânea de fumaça cinza-escura/preta e fumaça branca intensa.
3. **Ruídos de Fundo:** Presença de névoa/bruma atmosférica sobre as colinas à esquerda, gerando ruídos visuais com textura bastante semelhante à da fumaça rala.

---

## Impacto da Iluminação e do Contraste
Os fatores de iluminação solar e contraste afetaram diretamente o resultado do SAM 2:
* A forte luminosidade do céu claro esbranquiçado ao fundo reduziu drasticamente a diferença cromática (gradiente de luminância) em relação ao topo e às extremidades da pluma.
* Como a fumaça se torna mais clara e rarefeita à medida que se eleva, a semelhança de cor entre a fumaça e o céu faz com que o algoritmo perca a referência do contorno, interrompendo a máscara antes do limite real da fumaça.

---

## Conclusão e Justificativa das Melhorias
Todas as três configurações testadas demonstraram capacidade equivalente para delimitar a base e o tronco central mais denso da pluma sem provocar vazamentos (supersegmentação) para o céu ou terreno. Contudo, **nenhuma das três configurações conseguiu capturar a fumaça rala e difusa situada no lado direito**.

Esse resultado demonstra uma limitação do próprio modelo SAM 2 em lidar com objetos semitransparentes e de baixo contraste. Como o algoritmo se baseia fortemente em gradientes de cor e contornos bem definidos para delimitar objetos, a ausência de uma fronteira nítida entre a fumaça rala e o céu faz com que o ajuste por *prompts* (seja por caixa, pontos ou combinação híbrida) não seja suficiente para forçar o reconhecimento de bordas de baixíssima visibilidade.
