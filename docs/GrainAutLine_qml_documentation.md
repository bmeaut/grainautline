# GrainAutLine QML dokument�ci�

## A C++ �s a QML oldal kapcsolata

A QML bevezet�s�vel lehet�v� v�lt az alkalmaz�sok logik�j�nak �s felhaszn�l�i fel�let�nek a sz�tv�laszt�s�ra.
Azonban a C++-on alapul� programm�k�d�s, �s a Javascripten alapul� QML-en meg�rt GUI k�z�tti kapcsolat megl�t�hez tov�bbi l�p�sekre van sz�ks�g. Ez a r�sz azokat a szign�lokat, �s objektumokat �rja le, amiken kereszt�l ez a kapcsolat megval�sul, valamint a kapcsolat kialak�t�s�nak a m�dj�t is dokument�lja.

A kapcsolat kialak�t�s�nak a helye a `main.cpp`-ben tal�lhat� meg.

###  C++ objektumok el�rhet�v� t�tele a QML oldalon

A program jelenleg k�t objektumot oszt meg a felhaszn�l� fel�let sz�m�ra:

- `Logger`: Ez egy singleton napl�z� oszt�ly, amit LogWindow.qml modellk�nt haszn�l, hogy megjelen�tse a rendszernek a fontosabb esem�nyeit.
- `SlotManager` :Ennek az oszt�lynak h�rom tagv�ltoz�ja �rhet� el a QML oldalon:
	1. int `ActiveSlot`
	2. `QString OpenedFile`: Az �ppen megnyitott f�jlnak a nev�t tartalmazza, a QML oldalon csak arra van haszn�lva, hogy megn�zze �ppen van-e megnyitott f�jl.
	3. `bool ProcessorRunning`: Jelzi hogy van-e �ppen fut� process

Az objektumok megoszt�s�nak a m�dja a k�vetkez�:

	//main.ccp:
	
	//...
	context->setContextProperty(QStringLiteral("SlotManager"), &slotManager);
	//...
	context->setContextProperty(QStringLiteral("Logger"), &Logger::GetInstance());
	//...
El�rhet�v� tenni egy oszt�lynak a tagv�ltoz�it a `Q_PROPERTY()` makr� seg�ts�g�vel lehets�ges, r�szletek: 

 http://doc.qt.io/qt-5/qobject.html#Q_PROPERTY

Ezen k�v�l m�g k�t t�pus is be van regisztr�lva: az absztrakt `Processor` �s a `ProcessorSlot` oszt�ly. �gy ezeknek az oszt�lyoknak a tulajdons�gai el�rhet�ek a QML oldalon

### C++ �s QML szign�lok �sszekapcsol�sa

Az al�bbi QML szign�lok vannak �sszekapcsolva a SlotManager(ezen az oszt�lyon kereszt�l t�rt�nik a kommunik�ci�) megfelel� f�ggv�nyeivel:

- `keyPressed(int, int)`: A QML oldal a k�l�nb�z� gomblenyom�shoz tartoz� esem�nyeket el�sz�r lekezeli mag�nak, majd tov�bbdobja a C++ oldal sz�m�ra.
- Az al�bbi szign�lok jeleznek a SlotManager-nek ha valami t�rt�nik a v�szonnal:
	- canvasMousePressed(QString,QPointF,int,int,int)
	- canvasMouseReleased(QString,QPointF,int,int,int)
	- canvasMouseDoubleClicked(QString,QPointF,int,int,int)
	- canvasMouseMoved(QString,QPointF,int,int)
	- canvasSizeChanged(QString,QSizeF)
- fileOpened(QUrl): Jelez ha meg kell nyitni egy adott f�jlt.
- fileSavedAs(QUrl): Jelez, hogy az �ppen megnyitott f�jlt a param�terben megadott helyre kell lementeni
- `runTriggered()`: Ha el kell ind�tani egy folyamatot.

Ezen k�v�l m�g van egy szign�l ami az `ImageProvider` objektumhoz kapcsol�dik:
- renderComplete(QString)

### ImageProvider oszt�ly


## A QML oldal fel�p�t�se:

### main.qml

Ez az a f�jl amit a C++ oldalon bet�ltenek, �s k�zvetetten ez t�lti be a projekthez tartoz� egy�b QML f�jlokat. Itt tal�lhat� meg a men� �s az eszk�zt�r, valamint azok az Action-�k, amik az el�bbiekhez kellenek. Ezeken k�v�l a dial�gusablakok is itt vannak p�ld�nyos�tva:

- `LogWindow.qml`: Ezen kereszt�l lehet l�tni a program fontosabb esem�nyeit.
- `CheetSheet.qml`: Ez a lenyomott m�dos�t� (modifier) gombt�l f�gg�en ad seg�ts�get az �ppen el�rhet� gyorsbillenty�kh�z.
- 2 db FileDialog: egy a f�jl megnyit�s�hoz, egy f�jl ment�s�hez.
- 1 db MessageDialog ami az olyan funkci�k eset�n ugrik fel, amik m�g nincsenek megval�s�tva.

Minden egy�b elem ami a felhaszn�l�i fel�lethez tartozik, bele van rakva egy k�l�n f�jlba, a MainForm.qml-be, �gy �tl�that�bb a k�d. A main.qml-ben p�ld�nyos�tott MainForm elem �gy van kialak�tva, hogy mindig "f�kuszban" legyen, �gy minden egyes KeyEvent-et ez az elem kezel le miel�tt tov�bbadn� a C++ oldalnak. 

#### Process is running display

A programnak egyes m�veletei hosszabb ideig is eltarthatnak, �s a felhaszn�l� sz�m�ra sokszor nem egy�rtelm�, hogy �ppen csin�l-e valamit a program, vagy hogy a kiadott parancsot egy�ltal�n elkezdte-e v�grehajtani vagy sem. Ennek kik�sz�b�l�s�re annyiban m�dos�tva lett a GUI, hogy valah�nyszor �ppen valamilyen Processz fut, akkor azt egy forg� fogasker�kkel jelzi, az eszk�zt�rnak a jobbsz�l�n.

A fenti funkci� l�trehoz�s�hoz egy fogaskerekes k�pet �s egy hozz� tartoz� forgat� anim�ci�t kellet l�trehozni:

	    Image{ // Gear Wheel
                  id: gearWheelID
                  anchors.verticalCenter: parent.verticalCenter
                  anchors.right: parent.right
                  anchors.rightMargin: 6
                  visible: processorRunning
                  width: 18
                  height: 18
                  source: "qrc:///icons/gear.svg"

                  RotationAnimation on rotation{ //Spinning animation for the gear wheel
                    id: wheelAnimationID
                    running: processorRunning
                    loops: Animation.Infinite
                    from:0
                    to:360
                    duration: 1000
                  }
                }
Az anim�ci� elvileg folyamatosan ism�tli mag�t, de mind az anim�ci�, mind a k�p csak akkor jelenik meg, ha �ppen fut valamilyen folyamat, azaz `bool processorRunning` �rt�ke `true`.


![](images/run_display.png)

#### Run & Show process shortcuts

A k�nnyen kezelhet�s�g �rdek�ben sz�ks�g volt olyan gyorsbillenty�kre, amiknek a seg�ts�g�vel gyorsan le lehet futtatni az el�rhet� m�veleteket, valamint olyanokra amikkel gyorsan lehet v�ltogatni a kezel�fel�leten, az �ppen kijel�lt m�veletek k�z�tt. Ez �gy lett megoldva, hogy a `Ctrl+0-9` gombok lenyom�s�val kiv�laszt�dik a `ProcessorNamesBox`-nak a lenyomott sz�mnak megfelel� index� eleme. Ha csak sim�n megnyomjuk a `0-9` gombokat, akkor ezen fel�l m�g kiadja a `runTriggered()` szign�lt is, ezzel jelezve a `SlotManager`-nek, hogy az �ppen kijel�lt m�veletet hajtsa v�gre.

Ennek a megval�s�t�sa a `MainForm` elemnek a KeyEvent-eket kezel� f�ggv�ny�ben tal�lhat�:

	        Keys.onPressed:{
          // Run & Show shortcut
          var tmp = event.key - Qt.Key_0
          if( tmp>=0 && tmp<=9 ){
            showProcess(tmp)
            if(!(event.modifiers & Qt.ControlModifier))
              runTriggered()
          }
Ha a lenyomott gomb a `0-9` gombok valamelyike, akkor megh�vja a `showProcess(index)` f�ggv�nyt, ami  a forwardolt `processorNamesBox` elemnek az �ppen kiv�lasztott elem�t megv�ltoztatja, ezen fel�l ha a `Ctrl m�dos�t�` is le volt nyomva akkor m�g kiadja a `runTriggered()` szign�lt is.

	// main.qml:
    function showProcess(index){
      namesBox.currentIndex=index;
    }
	//...
	property alias namesBox: mainFormId.namesBox
	
	//MainForm.qml:

	//Forwarding processorNamesBox
	property var namesBox: processorNamesBox
	
#### Cheat Sheet

![](images/CheatSheet.png)

Ez az ablak ad inform�ci�t az �ppen el�rhet� gyorsbillenty�kr�l, a lenyomott m�dos�t� gombok f�ggv�ny�ben. A m�rete nem v�ltoztathat� a felhaszn�l� �ltal, automatikusan igazodik a az adott helyzetben el�rhet� gyorsbillenty�k sz�m�hoz.

Az eg�sz elemnek az alapja l�nyeg�ben egy `ListView` aminek a `model property` �rt�k�t dinamikusan v�ltoztatjuk a lenyomott m�dos�t� gomboknak megfelel�en. Az adatmodellek egy `Item` elemen bel�l tal�lhat�ak, a `rescoures property`-n bel�l, ami a nem vizu�lis lesz�rmazottaknak a list�ja. Ha csak egyszer�en az Item-en bel�l lenn�nek elhelyezve a modellek, akkor is ebben lenn�nek list�zva, de �gy hangs�lyosabb, hogy ennek az elemnek puszt�n a t�rol�s a c�lja.

	   Item{
      id: modelContainerID
      resources: [
        ListModel{
          id: model1
          // IF NOTHING IS PRESSED:
          ListElement{description: "Clears the entire Aux" ; shortcut: "Del"}
          //...
          ListElement{description: "auto-select"; shortcut: "Right Click" }
          ListElement{description: "Run dedicated processors"; shortcut: "1...9" }
        },
        ListModel{
          id: model2
          // IF Ctrl IS PRESSED:
          ListElement{description: "Erasing" ; shortcut: "Ctrl + Lclick"}
          //...
          ListElement{description: "Save As"; shortcut: "Ctrl+Shift+S" }
          ListElement{description: "Open"; shortcut: "Ctrl+O" }
        }
      ]
    }
   A megold�snak az el�nye, hogy a b�v�t�se k�nnyen megoldhat�: ha p�ld�ul azt szeretn�nk, hogy az `Alt` gomb lenyom�s�ra is reag�ljon, akkor csak hozz� kell adni egy �jabb modellt, amiben az Alt gombhoz tartoz� gyorsbillenty�ket felsoroljuk.
  
 A modellnek a cser�j�t a `change_model(index)` f�ggv�ny v�gzi:

	// CheatSheet.qml:
	   function change_model(index){
    list1_ID.model = modelContainerID.resources[index]
    
	    rootID.minimumHeight = 0;
	    rootID.maximumHeight = margins * 2 + list1_ID.model.count * (delegate_height + spacing) - spacing
	    rootID.height = maximumHeight
	    rootID.minimumHeight = maximumHeight
	    }
    }
A f�ggv�nyen bel�l az�rt van sz�ks�g az ablaknak a minimum �s maximum �rt�k�nek az �ll�tgat�s�ra, mert az ablaknak a m�rete �gy lett megoldva, hogy a minimum �s a maximum �rt�k mindig a pillanatnyi �rt�kre �ll�t�dik. Azonban ezzel az a gond, hogy a pillanatnyi �rt�ket csak a minimum �s maximum k�z�tti �rt�kre lehet be�ll�tani, ez�rt ha v�ltoznia kell a pillanatnyi �rt�knek a m�dosult tartalom miatt, akkor el�sz�r a minimumot �s maximumot �gy kell be�ll�tani, hogy a pillanatnyi �rt�k megv�ltoztathat� legyen.

Mivel minden KeyEvent-et a `main.qml`-ben p�ld�nyos�tott `MainForm` elem kap meg - k�sz�nhet�en annak, hogy mindig f�kuszban van -,  ez�rt a `CheatSheet`-hez tartoz� esem�nyeket is itt kell lekezelni:

	    MainForm {
        id: mainFormId
        //...
        Keys.onPressed:{
        //...
        if(event.modifiers & Qt.ControlModifier)
            chiChiId.change_model(1)
          else
            chiChiId.change_model(0)
        //...
        }
        // For the Cheat Sheet
        Keys.onReleased:{
          if(event.modifiers & Qt.ControlModifier) chiChiId.change_model(1)
          else                                     chiChiId.change_model(0)
        }
        
    }//MainForm

### MainForm.qml

![](images/MainForm.png)

A men�t �s az eszk�zt�rat lesz�m�tva a f�ablaknak minden grafikus eleme ebben a f�jlban van p�ld�nyos�tva, nevezetesen:

- `PsdCanvas`: Ez a v�szna a programnak amire rajzolni lehet, �s amin megjelennek a k�l�nb�z� Aux r�tegek.
-  `ProcessorNamesBox`: Egy leny�l� lista, amin kereszt�l ki lehet v�lasztani az �ppen v�grehajtand� m�veletet. A `SlotManager`-t �s a `ProcessorSlot`-ot haszn�lja adatmodellnek, �s t�le k�rdezi le az el�rhet� m�veleteknek a nev�t
- `ProcessorPropertiesBox`: Az �ppen kiv�lasztott m�velethez tartoz� be�ll�t�sokat jelen�ti meg. Ez a `SlotManager`-t a `ProcessorSlot`-ot valamint `Processor`-t haszn�lja adatmodellk�nt, �s t�l�k k�rdezi le az adott m�velethez tartoz� funkci�kat.
- `SupplementaryCanvasBox`: Ez a `PsdCanvas`-nak egy m�dos�t�sa, ami arra haszn�latos, hogy seg�tsen a t�j�koz�d�sban rajzol�s k�zben
- `LayerBox`: Ezen kereszt�l lehet �ll�tani, hogy �ppen melyik r�teg legyen l�that�, �s milyen fok� �ttetsz�s�ggel. A `SlotManager`-t �s a `ProcessorSlot`-ot haszn�lja adatmodellk�nt.

### PsdCanvas.qml

Ez tulajdonk�ppen egy `Image` elemb�l �ll ami a v�sznat alkotja meg, amire rajzolni lehet. A v�szon nagy�t�snak �s mozgat�s�nak megval�s�t�s�hoz ez az elem m�g rendelkezik �tsk�l�z� �s mozgat� transzform�ci�kkal: `imageScaleTransformID`, `imageTranslateTransformID`. Ezeken kereszt�l lehet a k�pen v�grehajtani a megfelel� transzform�ci�kat.

#### PsdCanvas.qml fel�p�t�se

![](images/PsdCanvas.png)

K�t k�l�nb�z� `MouseArea` tal�lhat� a `PsdCanvas`-ban:

- Az els� az folyamatosan k�veti m�retben �s poz�ci�ban a v�sznat, �s a C++ oldallal tart kapcsolatot: Jelez ha b�rmilyen ha valamilyen esem�ny t�rt�nik a v�szonnal (`canvasMouseDoubleClicked`, `canvasMouseReleased`, stb...)
- A m�sodik a teljes teret kit�lti, �s ez kezeli a v�szonnak a nagy�t�s�t �s mozgat�s�t.

#### Zoom in & Zoom out

	  MouseArea {
        id: areaID
        //...
        onWheel: {
            //var imageScale = imageScaleTransform.scale / 15 // for smooth scaling 
            var point = mapToItem(imageID, wheel.x, wheel.y);

            if (wheel.angleDelta.y > 0) {
                zoom(imageContainer.zoom_in, point);
            } else {
                zoom(imageContainer.zoom_out, point);
            }
        }
        //...
A g�rget�s sor�n a kurzor pozici�j�n�l l�v� k�ppontot helyben hagyja, �gy �rz�dik term�szetesnek a haszn�lata. Azonban minden vizu�lis elemnek megvan a saj�t koordin�tarendszere, �s ha a k�t elem nincs pont ugyanabban a poz�ci�ban, akkor a `MouseArea` esem�ny�b�l vett koordin�ta nem egyezik meg a kurzor �ltal mutatott k�ppont koordin�tj�val, ez�rt ezt �t kell transzform�lni. Ezt a c�lt szolg�lja a `mapToItem()` f�ggv�ny, ami a kurzor koordtin�t�ir�l megmondja, hogy az  `Image` elemnek a koordin�tarendszer�ben hol helyezkedik el.
A zoomol�st a `zoom(direction, point)` f�ggv�ny v�zi el:

	function zoom(direction, point){
      imageID.scale_img(direction);

      if(typeof(point)==='undefined')
        point = areaID.mapToItem(imageID, center_canvas.x, center_canvas.y);

      var scale_dif = imageScaleTransformID.scale - imageScaleTransformID.prev_scale;
      imageTranslateTransformID.x -= point.x * scale_dif;
      imageTranslateTransformID.y -= point.y * scale_dif;
    }
A zoomol�shoz el�sz�r is a k�pet �t kell sk�l�zni (scale_img f�ggv�ny), azonban ahhoz hogy a k�v�nt pont helyben maradjon (point koordin�ta), ahhoz el is kell tolni valamennyivel. Ha point nincs megadva, akkor automatikusan a k�perny� k�zep�t hagyja helyben, ami az�rt lehet fontos, mert az eszk�zt�rb�l is lehet nagy�tani, �s ebben az esetben nincs megadott koordin�ta.

#### Fit to Page

Ez a funkci� pont annyira nagy�tja ki  a k�pet, hogy teljesen kif�rjen a v�szonra:

	    function fitToPage(){
      var scale_w = areaID.width / imageID.sourceSize.width;
      var scale_h = areaID.height / imageID.sourceSize.height;

      if(scale_w < scale_h) imageScaleTransformID.scale = scale_w;
      else                  imageScaleTransformID.scale = scale_h;

      imageTranslateTransformID.x = 0;
      imageTranslateTransformID.y = 0;
    }
 L�nyeg�ben megn�zi, hogy a k�p forr�s�nak a magass�g�t �s sz�less�g�t v�ve melyik az a legkisebb sz�m, amelyikn�l az egyik akkora lesz mint a teljes v�szonnak a megfelel� m�rete. Az �tsk�l�z�s ut�n pedig az eltol�sokat lenull�zza, �gy pont a bal f�ls� sarokban lesz a k�p.

![](images/FitToPage.png)

### ProcessorNamesBox.qml

Ebben a leny�l� men�ben jelennek meg a kiv�laszthat� m�veletek nev�nek a list�ja. Mindig tudatja a `SlotManager`-rel, ha v�ltozik a jelenleg kiv�lasztott m�velet, �gy ha a `runTriggered()` szign�l ki van adva, akkor mindig a megfelel� `Process` lesz futtatva. A Run & show shortcuts ezt haszn�lja ki, �gy l�nyeg�ben csak megv�ltoztatja a kiv�lasztott m�veletet, �s ha kell akkor kiadja a `runTriggered()` szign�lt.

