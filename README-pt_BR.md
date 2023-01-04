# Auto Synced & Translated Dubs
Traduz automaticamente o texto de um vídeo em idiomas escolhidos com base num ficheiro de legendas, e também usa a voz AI para dublar o vídeo, enquanto o mantém devidamente sincronizado com o vídeo original usando os tempos das legendas.
 
### Como Funciona
Se já tiver um ficheiro de legendas SRT feito por humanos para um vídeo, este irá:
1. Use o Google Cloud para traduzir automaticamente o texto, e criar novos ficheiros SRT traduzidos
2. Criar clips áudio texto-fala do texto traduzido (utilizando vozes neurais mais realistas)
3. Utilize os tempos das linhas de legenda para calcular a duração correcta de cada clipe de áudio falado
4. Esticar ou encolher o vídeo traduzido para ter exatamente o mesmo comprimento que o discurso original, e inserido no mesmo ponto do áudio. Portanto, o discurso traduzido permanecerá perfeitamente em sincronia com o vídeo original.
    - Opcional (On by Default): Em vez de esticar os clipes de áudio, pode, em vez disso, fazer uma segunda passagem, sintetizando cada clip através do API, utilizando a velocidade de fala apropriada calculada durante a primeira passagem. Isto melhora drasticamente a qualidade de áudio.
    
### Características chave adicionais
- Cria versões traduzidas do ficheiro de legendas do SRT
- Processamento em lote de múltiplos idiomas em sequência
- Configurar ficheiros para guardar a tradução, síntese, e definições de idioma para reutilização
- Script incluído para adicionar todas as faixas de áudio de todas as línguas a um ficheiro de vídeo
   - Com capacidade para fundir uma faixa de efeitos sonoros em cada faixa linguística
- Inclui guião para tradução de um título e descrição de vídeo do YouTube para várias línguas


----

# Instruções

### Requisitos externos:
- ffmpeg deve ser instalado (https://ffmpeg.org/download.html)
- Vai precisar dos binários para um programa chamado 'rubberband' ( https://breakfastquay.com/rubberband/ ) . Não precisa de ser instalado, basta colocar os dois exe's e o ficheiro dll no mesmo diretório que os scripts.

## Instalação e Configuração
1. Descarregar ou clonar o repo e instalar os requisitos utilizando `pip install -r requirements.txt`.
   - Escrevi isto usando o Python 3.9 mas provavelmente também funcionará com versões anteriores
2. Instalar os programas mencionados nos 'Requisitos Externos' acima.
3. Instale o seu Google Cloud (Ver Wiki) e/ou o acesso API do Microsoft Azure, e defina as variáveis em `cloud_service_settings.ini`. 
   - Recomendo o Azure para o sintetizador de voz TTS porque na minha opinião têm vozes mais novas e melhores, e em maior qualidade (o Azure suporta taxa de amostragem até 48KHz vs 24KHz com o Google). 
- O Google Cloud é atualmente a única opção para a parte da tradução de texto.
4. Configure as suas configurações em `config.ini`. As configurações padrão devem funcionar na maioria dos casos, mas leia-as especialmente se estiver a utilizar o Azure para TTS porque há mais opções aplicáveis que pode querer personalizar.
   - Esta configuração inclui opções tais como a capacidade de saltar a tradução de texto, definir formatos e taxa de amostragem, e utilizar o sintetizador de voz de duas passagens
5. Finalmente, abrir `batch.ini` para definir o idioma e as definições de voz que serão utilizadas para cada execução. 
- Na secção superior `[SETTINGS]` introduzirá o caminho para o ficheiro de vídeo original (utilizado para obter o comprimento de áudio correto), e o caminho do ficheiro de subtítulos original
   - Também pode utilizar a variável `enabled_languages' para listar todas as línguas que serão traduzidas e sintetizadas de uma só vez. Os números corresponderão às secções `[LANGUAGE-#]` no mesmo ficheiro de configuração. O programa processará apenas os idiomas listados nesta variável.
   - Isto permite-lhe adicionar tantos idiomas predefinidos quantos desejar (como a voz preferida por idioma), e pode escolher que idiomas pretende utilizar (ou não utilizar) para uma determinada execução.

## Instruções de utilização
- **Como correr:** Após configurar os ficheiros de configuração, basta correr o script main.py utilizando `python main.py` e deixá-lo correr até à conclusão
   - Os ficheiros de legendas e faixas áudio dubladas resultantes serão colocados numa pasta chamada 'output'.
- **Opcional:** Pode utilizar o script separado `TrackAdder.py` para adicionar automaticamente as faixas de linguagem resultantes a um ficheiro de vídeo mp4. Requer ffmpeg para ser instalado.
   - Abra o ficheiro de script com um editor de texto e altere os valores na secção "User Settings" no topo.
   - Isto irá rotular as faixas para que o ficheiro de vídeo esteja pronto para ser carregado no YouTube. ENTÃO, a funcionalidade de múltiplas faixas de áudio só está disponível para um número limitado de canais. Muito provavelmente precisará de contactar o apoio ao criador do YouTube para pedir acesso, mas não há garantias de que o concedam.
- **Opcional:** Pode utilizar o script separado `TitleTranslator.py` se carregar no YouTube, que lhe permite introduzir o Título e Descrição de um vídeo, e o texto será traduzido para todas as línguas habilitadas em `batch.ini`. Serão colocados juntos num único ficheiro de texto na pasta "output".

----

## Notas adicionais:
- Isto funciona melhor com legendas que não removam os espaços entre frases e linhas.
- Por enquanto, o processo pressupõe apenas a existência de um orador. No entanto, se conseguir fazer ficheiros SRT separados para cada orador, poderá gerar cada faixa TTS separadamente usando vozes diferentes, depois combiná-las posteriormente.
- Só suporta Google Translate API para tradução de texto, mas suporta tanto Google como Azure para Text-To-Speech com vozes neurais.
- Este guião foi escrito com o meu próprio fluxo de trabalho pessoal em mente. Ou seja:
- Uso [**OpenAI Whisper**](https://github.com/openai/whisper) para transcrever os vídeos localmente, depois uso [**Descript***](https://www.descript.com/) para sincronizar essa transcrição e retocar com as correções.
    - Depois exporto o ficheiro SRT com Descript, o que é ideal, porque não se limita a marcar as horas de início e fim de cada linha de subtítulo uma ao lado da outra. Isto significa que o dub resultante irá preservar as pausas entre frases do discurso original. Se utilizar legendas de outro programa, pode achar que as pausas entre linhas são demasiado curtas.
    - As definições de exportação da SRT em Descript que parecem funcionar decentemente para dublagem são *150 caracteres máximos por linha*, e *1 linha máxima por cartão*.
- A função de sintetização "Two Pass" (pode ser ativada na configuração) melhorará drasticamente a qualidade do resultado final, mas exigirá a sintetização de cada clip duas vezes, duplicando assim qualquer custo API.

----

### Para exemplos de resultados Ver: [Página Wiki de Exemplos](https://github.com/ThioJoe/Auto-Synced-Translated-Dubs/wiki/Examples)
### Para características planeadas Ver: [Página Wiki de Características Planeadas](https://github.com/ThioJoe/Auto-Synced-Translated-Dubs/wiki/Planned-Features)
### Para instruções de configuração do Projeto Google Cloud Ver: [Página Wiki de Instruções](https://github.com/ThioJoe/Auto-Synced-Translated-Dubs/wiki/Instructions:-Obtaining-an-API-Key)
### Para instruções de configuração do Microsoft Azure Ver: [Página Wiki de Instruções Azure](https://github.com/ThioJoe/Auto-Synced-Translated-Dubs/wiki/Instructions:-Microsoft-Azure-Setup)



-- Traduzido com a versão gratuita do tradutor - www.DeepL.com/Translator
