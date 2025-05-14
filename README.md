# Criando Menus de Serviço Dolphin
## Aprenda a criar menus de serviço para o Dolphin.
- A capacidade de selecionar ações específicas do tipo mime no menu de contexto de um gerenciador de arquivos do KDE é um recurso frequentemente solicitado. A agradável surpresa é que isso já é possível. A surpresa ainda mais agradável é que você não precisa ser um desenvolvedor de software para criar novas ações. Este artigo detalha passo a passo como adicionar novas ações rápida e facilmente aos menus de contexto do gerenciador de arquivos do KDE.

## Introdução
- No idioma do KDE, um “menu de serviço” é uma entrada especial que aparece em um menu de contexto (ou outras interfaces baseadas em contexto) para um arquivo (ou para um diretório), dependendo do tipo de arquivo selecionado.

- Por exemplo, se você tiver o utilitário de arquivamento de arquivos do KDE Ark instalado, você verá uma entrada de menu para `“Extrair aqui…”` sempre que clicar com o botão direito do mouse em um arquivo de arquivo. A opção `“Extrair aqui…”` é um menu de serviço.

- Criar novos menus de serviço é muito simples, exigindo nada mais do que uma ideia e um editor de texto. Você não precisa ser um programador ou um assistente do KDE para fazê-los. Neste tutorial, estaremos criando um conjunto de ações que nos permite definir uma imagem como papel de parede da área de trabalho apenas clicando com o botão direito do mouse e selecionando `“Usar como papel de parede”`. Ao final deste tutorial, você deverá ser capaz de criar seus próprios menus de serviço com facilidade.

## Onde os menus de serviço estão localizados
- Os menus de serviço são definidos usando arquivos .desktop, que são os mesmos tipos de arquivos usados ​​para criar entradas no inicializador do Plasma. Esses arquivos de menu de serviço são encontrados em

Subpasta ServiceMenus de uma das pastas que podem ser determinadas com
```
kf5-config --path services
```
No caso da minha máquina doméstica, isso significa que os arquivos do menu de serviço podem ser encontrados nos seguintes locais:
```
~/.local/share/kservices5/ServiceMenus/   
```
```
/usr/share/kservices5/ServiceMenus/
```
### O início do menu de serviço
- Começaremos a criar nosso menu de serviço de papel de parede escolhendo um nome para o arquivo: `setAsWallpaper.desktop` soa bem. A única coisa que realmente importa em relação ao nome é que ele é único e termina com `.desktop`. Em seguida, vamos abrir o arquivo em um editor de texto. A primeira coisa que colocaremos no arquivo é a seção `“Desktop Entry”`:
```
[Desktop Entry]
Type=Service
X-KDE-ServiceTypes=KonqPopupMenu/Plugin
MimeType=image/png;image/jpeg;
Actions=setAsWallpaper
Todo arquivo de menu de serviço deve ter essas quatro linhas. Vamos examinar cada uma dessas linhas uma de cada vez.

[Desktop Entry]
Os arquivos de configuração do KDE, incluindo os arquivos .desktop, separam as configurações individuais em seções. Uma seção começa com um título composto por letras, números e espaços entre colchetes em uma linha por si só. Esta primeira linha significa que todas as opções que seguem, até o próximo título, pertencem ao grupo “Desktop Entry”.

Type=Service
X-KDE-ServiceTypes=KonqPopupMenu/Plugin
MimeType=image/png;image/jpeg;
```
A primeira linha indica que este arquivo `.desktop` é do tipo Service; isso é necessário, pois o tipo padrão é `Application` (ou seja, algo com uma linha Exec). Serviço é qualquer outra coisa, como plugins.

A entrada ServiceTypes refere-se ao tipo de plug-in que este arquivo da área de trabalho define e especifica que na verdade é um menu de serviço Dolphin.

A linha `MimeType` define o tipo de arquivo ao qual este menu de serviço se aplica. Você pode definir mais de um tipo `MIME` fornecendo uma lista separada por ponto e vírgula (mas sem espaços). Nesse caso, nosso menu de serviço aparecerá quando selecionarmos imagens `PNG` ou `JPEG`. O painel de controle Associações de Arquivos é um bom lugar para procurar definições de tipo mime.

## Dica
- Para criar um menu de serviço para diretórios, use o `inode/directory mimetype`. Para criar um menu de serviço para todos os arquivos, use o `application/octet-stream mimetype` base
- Você também pode especificar um grupo inteiro de mimetypes usando `“typeglobs”`. Para fazer com que nosso menu de serviço se aplique não apenas a `PNGs` e `JPEGs`, mas a todas as imagens, basta alterar a entrada `ServiceTypes` para:
```
X-KDE-ServiceTypes=KonqPopupMenu/Plugin
MimeType=image/*;
```
Agora, quando clicamos com o botão direito do mouse em qualquer arquivo de imagem no Dolphin, podemos selecioná-lo como plano de fundo.
```
Actions=setAsWallpaper
```
A entrada Actions define as ações que criaremos em nosso servicemenu. Assim como nos `ServiceTypes`, você pode definir mais de uma ação usando uma lista separada por ponto e vírgula. Cada uma das ações listadas terá uma seção própria definindo o que essa ação faz. Na verdade, esse é o nosso próximo passo.

### Criando uma ação
- Até agora, definimos uma ação em nosso arquivo de menu de serviço: `setAsWallpaper`. Agora precisamos definir como essa ação se parece e o que ela realmente faz. Começamos adicionando um novo cabeçalho ao final do nosso arquivo:
```
[Desktop Action setAsWallpaper]
```
Observe que ele contém o nome da ação `setAsWallpaper`. É importante observar que esses arquivos diferenciam maiúsculas de minúsculas, portanto, precisamos observar a capitalização aqui. Agora que temos uma seção para nossa ação, vamos dar à nossa ação um nome que o usuário verá.
```
Name=Set As Background Image
```
Para traduzir o nome, adicionamos outra entrada de Nome seguida do código do idioma. Por exemplo, a tradução finlandesa para o serviço `“Definir como imagem de fundo”` é fornecida por uma entrada parecida com esta:
```
Name[fi]=Aseta taustakuvaksi
```
Em seguida, vamos adicionar um ícone:
```
Icon=background
```
Observe que não incluímos a extensão de arquivo `.png`, mas apenas nos referimos ao ícone pelo nome. Você pode encontrar ícones pelo nome procurando `/usr/share/icons/` ou instalando `plasma-sdke` usando o aplicativo `Cuttlefish` . Se tivéssemos deixado a Iconlinha de fora, nossa ação ainda funcionaria, só não pareceria tão chique. Agora que alcançamos o capricho, vamos terminar tornando-o útil:
```
Exec=qdbus org.kde.plasmashell /PlasmaShell org.kde.PlasmaShell.evaluateScript 'var allDesktops = desktops();print (allDesktops);for (i=0;i<allDesktops.length;i++) {d = allDesktops[i];d.wallpaperPlugin = "org.kde.image";d.currentConfigGroup = Array("Wallpaper", "org.kde.image", "General");d.writeConfig("Image", "%u")}';
```
A linha `Exec` define o que é executado quando o usuário seleciona a ação no menu. Podemos colocar qualquer comando que quisermos lá. A mágica nesta linha é o `“%u”` que é substituído pelo URL do arquivo de imagem antes que o comando seja executado. Se nosso comando puder aceitar mais de um arquivo por vez, podemos usar `“%U”` em vez disso. Existem outros %valores especiais, mas `%u` e `%U` são provavelmente os mais úteis para menus de serviço.

Nosso arquivo agora está assim:
```
[Desktop Entry]
Type=Service
X-KDE-ServiceTypes=KonqPopupMenu/Plugin
MimeType=image/*;
Actions=setAsWallpaper

[Desktop Action setAsWallpaper]
Name=Use As Wallpaper
Icon=background
Exec=qdbus org.kde.plasmashell /PlasmaShell org.kde.PlasmaShell.evaluateScript 'var allDesktops = desktops();print (allDesktops);for (i=0;i<allDesktops.length;i++) {d = allDesktops[i];d.wallpaperPlugin = "org.kde.image";d.currentConfigGroup = Array("Wallpaper", "org.kde.image", "General");d.writeConfig("Image", "%u")}';
```
Se o salvarmos e abrirmos o Dolphin, ao clicarmos com o botão direito do mouse em uma imagem PNG, JPEG ou GIF, teremos agora um item `“Definir como plano de fundo”` no menu. Experimente!

### Dica
- Se você tiver uma tarefa complexa que requer mais de um comando __(por exemplo, se quisermos copiar o arquivo de imagem em algum lugar primeiro e depois usar o `D-Bus` para defini-lo como plano de fundo)__, use um shell:
```
Exec=/bin/sh -c ";<YOUR COMMANDS HERE>"
```
E agora de volta à nossa transmissão programada regularmente…
De volta da terra do `D-Bus`, produzimos um menu de serviço funcional. O que agora? Nós melhoramos, é claro!

Nosso menu de serviço atual dimensiona a imagem para o tamanho da área de trabalho e a define como papel de parede. Mas isso não é apropriado para azulejos de papel de parede que não são dimensionados, mas devem ser, bem, lado a lado. Então vamos adicionar uma ação para tiles. Primeiro, precisaremos alterar a linha Actions para dizer algo assim:
```
Actions=setAsWallpaper;tileAsWallpaper;
```
Observe o uso de ponto e vírgula em vez de vírgulas nessa linha. Em seguida, adicionaremos uma nova seção de ação ao final do arquivo que se parece com isso:
```
[Desktop Action tileAsWallpaper]
Name=Use As Wallpaper Tile
Icon=background
Exec=qdbus org.kde.plasmashell /PlasmaShell org.kde.PlasmaShell.evaluateScript 'var allDesktops = desktops();print (allDesktops);for (i=0;i<allDesktops.length;i++) {d = allDesktops[i];d.wallpaperPlugin = "org.kde.image";d.currentConfigGroup = Array("Wallpaper", "org.kde.image", "General");d.writeConfig("FillMode", "3")};d.writeConfig("Image", "%u")}';
```
Observe que `“tileAsWallpaper”` aparece no título da seção de ação. Isso é o que diz ao Dolphin qual é a ação. Além disso, temos um `Name` ligeiramente diferente e uma linha `Exec` muito ligeiramente diferente. Agora, quando clicamos com o botão direito do mouse em uma imagem, temos outra opção, desta vez para colocar a imagem lado a lado em nossa área de trabalho. Nós nem tivemos que reiniciar o Dolphin, já que ele relê automaticamente o arquivo quando ele muda!

A área de trabalho do Plasma oferece várias opções de imagem de fundo, das quais `Scale` e `Tile` são apenas duas. Claro, se começarmos a adicionar todas essas várias opções de plano de fundo, e depois adicioná-las a todos os outros menus de serviço que uma instalação típica do KDE possui, é fácil ver como o menu Ação pode rapidamente sair do controle. Podemos criar submenus para nossos menus de serviço adicionando uma linha como a seguinte ao:
```
[Desktop Entry]
grupo do arquivo .desktop:
X-KDE-Submenu=Set As Background
```
Isso criará um submenu chamado `“Definir como plano de fundo”` e colocará todas as ações do menu de serviço nele. Nosso arquivo `.desktop` do menu de serviço agora se parece com isso:
```
[Desktop Entry]
Type=Service
X-KDE-ServiceTypes=KonqPopupMenu/Plugin
MimeType=image/*;
Actions=setAsWallpaper;tileAsWallpaper;
X-KDE-Submenu=Use As Wallpaper

[Desktop Action setAsWallpaper]
Name=Scaled
Icon=background
Exec=qdbus org.kde.plasmashell /PlasmaShell org.kde.PlasmaShell.evaluateScript 'var allDesktops = desktops();print (allDesktops);for (i=0;i<allDesktops.length;i++) {d = allDesktops[i];d.wallpaperPlugin = "org.kde.image";d.currentConfigGroup = Array("Wallpaper", "org.kde.image", "General");d.writeConfig("Image", "%u")}';

[Desktop Action tileAsWallpaper]
Name=Tiled
Icon=background
Exec=qdbus org.kde.plasmashell /PlasmaShell org.kde.PlasmaShell.evaluateScript 'var allDesktops = desktops();print (allDesktops);for (i=0;i<allDesktops.length;i++) {d = allDesktops[i];d.wallpaperPlugin = "org.kde.image";d.currentConfigGroup = Array("Wallpaper", "org.kde.image", "General");d.writeConfig("FillMode", "3")};d.writeConfig("Image", "%u")}';
```
### Propriedades avançadas
- Esta é uma coleção de propriedades avançadas. Você pode não precisar deles para o seu projeto, mas é útil saber que eles existem.

### Propriedade	Explicação
- `Protocolo X-KDE`	Requer que o esquema das URLs seja igual, por exemplo em um compartilhamento de samba isso seria smb
- `Protocolos X-KDE`	Requer que o esquema dos URLs esteja contido na lista, por exemplo:
`file,smb`

- `X-KDE-RequiredNumberOfUrls`	Número de URLs que podem ser selecionados para que este menu seja exibido. Para permitir várias combinações, você pode separar os números com uma vírgula, por exemplo: `2,4,6`

- `X-KDE-MinNumberOfUrls`	Número mínimo de URLs que podem ser selecionados para que este menu seja exibido. Esta propriedade está disponível desde a versão: `5.76`
- `X-KDE-MaxNumberOfUrls`	Número máximo de URLs que podem ser selecionados para que este menu seja exibido. Esta propriedade está disponível desde a versão: `5.76`

___Se você precisar de opções mais dinâmicas, confira `KAbstractFileItemActionPlugin` sobre como escrever esses plugins em `C++`___.

### Exemplos
- Esta é uma lista de menus de serviço contribuídos pelo usuário. Sinta-se à vontade para adicionar seus próprios menus de serviço personalizados aqui, se achar que eles podem ser usados ​​por outras pessoas.

Menus de serviço KDE da Z-Ray Entertainment
