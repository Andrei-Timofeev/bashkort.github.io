<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta
      name="viewport"
      content="initial-scale=1,maximum-scale=1,user-scalable=no"
    />
    <title>Map_Bashkortostan</title>

    <link
      rel="stylesheet"
      href="https://js.arcgis.com/4.14/esri/themes/dark/main.css"
    />
    <script src="https://js.arcgis.com/4.14/"></script>

    <style>
      html,
      body,
      #viewDiv {
        padding: 0;
        margin: 0;
        height: 100%;
        width: 100%;
      }

      #seasons-filter {
        height: 410px;
        width: 100%;
        visibility: hidden;
      }

      .season-item {
        width: 100%;
        padding: 8x;
        text-align: center;
        vertical-align: baseline;
        cursor: pointer;
        height: 40px;
      }

      .season-item:focus {
        background-color: dimgrey;
      }

      .season-item:hover {
        background-color: dimgrey;
      }

      #titleDiv {
	  position: absolute;
		margin-top: 15px;
        top: 0;
        left: 0;
		margin-left:60px;
        padding: 10px;
		font-size: 11px;
      }

      #titleText {
        font-size: 10pt;
        font-weight: 60;
        padding-bottom: 10px;
		
      }
	   #about_author {
        padding: 8px;
        background-color: black;
        width: 400px;
      }
	 
	  .panel-container {
        position: relative;
        width: 100%;
        height: 100%;
      }

      .panel-side {
        padding: 2px;
        box-sizing: border-box;
        width: 300px;
        height: 30%;
        position: absolute;
		margin-top: 186px;
        top: 0;
        right: 0;
		margin-right:10px;
        color: #fff;
        background-color: rgba(0, 0, 0, 0.6);
        overflow: auto;
        z-index: 60;
      }

      .panel-side h3 {
        padding: 0 20px;
        margin: 20px 0;
      }

      .panel-side ul {
        list-style: none;
        margin: 0;
        padding: 0;
      }

      .panel-side li {
        list-style: none;
        padding: 10px 20px;
      }

      .panel-result {
        cursor: pointer;
        margin: 2px 0;
        background-color: rgba(0, 0, 0, 0.3);
      }

      .panel-result:hover,
      .panel-result:focus {
        color: orange;
        background-color: rgba(0, 0, 0, 0.75);
      }
    </style>
    <script>
      require([
        "esri/views/MapView",
        "esri/Map",
        "esri/layers/FeatureLayer",
        "esri/widgets/Expand",
		"esri/widgets/Home",
		"esri/widgets/Legend",
		"esri/widgets/Expand",
		"esri/widgets/Search",
      "esri/layers/FeatureLayer"
      ], function(MapView, Map, FeatureLayer, Expand, Home, Legend, Expand, Search, FeatureLayer) {
        
		let floodLayerView;
        // flash flood warnings layer
        const layer = new FeatureLayer({
          portalItem: {
            id: "398865c5759a46958fb5cadb13983b26"
          },
          outFields: ["form"]
        });
		
		const layerRegion = new FeatureLayer({
          portalItem: {
            id: "9f7d3ddc986f440580cb524e68cd640f"
          },
		  	opacity: 0.8
		  });
			
        const map = new Map({
          basemap: "gray-vector",
          layers: layer
        });
		map.add(layerRegion);
		map.add(layer);
        const view = new MapView({
          map: map,
          container: "viewDiv",
          center: [55.97829,54.73596],
          zoom: 6
        });
		 var homeBtn = new Home({
          view: view
        });
		
        const seasonsNodes = document.querySelectorAll(`.form`);
        const seasonsElement = document.getElementById("seasons-filter");

        // click event handler for seasons choices
        seasonsElement.addEventListener("click", filterBySeason);

        // User clicked on Winter, Spring, Summer or Fall
        // set an attribute filter on flood warnings layer view
        // to display the warnings issued in that season
        function filterBySeason(event) {
          const selectedSeason = event.target.getAttribute("form");
          floodLayerView.filter = {
            where: "form = '" + selectedSeason + "'"
          };
        }
        view.whenLayerView(layer).then(function(layerView) {
          floodLayerView = layerView;

          // set up UI items
          seasonsElement.style.visibility = "visible";
          const seasonsExpand = new Expand({
            view: view,
            content: seasonsElement,
            expandIconClass: "esri-icon-filter",
            group: "top-left",
          });
          //clear the filters when user closes the expand widget
          seasonsExpand.watch("expanded", function() {
            if (!seasonsExpand.expanded) {
              floodLayerView.filter = null;
            }
          });
		
		const legend = new Expand({
          content: new Legend({
            view: view,
            style: "card" // other styles include 'classic'
          }),
          view: view,
          expanded: false
        });
		
		const about_author = document.createElement("div");
          about_author.id = "about_author";
          about_author.innerHTML =
            "Отображение крупнейших сельхозхозяйственных предприятий на территории Республики Башкортостан. Проект был выполнен студентами группы ИСТ-308д: Тимофеевым А.В. и Бершаком А.И.";

          const aboutExpand = new Expand({
            expandIconClass: "esri-icon-question",
            expandTooltip: "Сведения об авторах",
            view: view,
            content: about_author,
            expanded: false
          });
		  
		  
		  var search = new Search({
			view: view
			
			})
 
  // Find address
        search.sources.push({
        layer: layer,
        searchFields: ["Name_M"],
        displayField: "Name_M",
        exactMatch: false,
        outFields: ["Name_M", "address"],
        resultGraphicEnabled: true,
        name: "Предприятия",
        placeholder: "Например: ОАО «Базис»",
      });
		
	//ПЕРЕМЕЩЕНИЕ РЕГИНОНОВ ОТ 
	const listNode = document.getElementById("nyc_graphics");
	 
	let graphics;

        view.whenLayerView(layerRegion).then(function(layerView) {
          layerView.watch("updating", function(value) {
            if (!value) {
              // wait for the layer view to finish updating

              // query all the features available for drawing.
              layerView
                .queryFeatures({
                  geometry: view.extent,
                  returnGeometry: true,
                  orderByFields: ["Name"]
                })
                .then(function(results) {
                  graphics = results.features;

                  const fragment = document.createDocumentFragment();

                  graphics.forEach(function(result, index) {
                    const attributes = result.attributes;
                    const name = attributes.Name;

                    // Create a list zip codes in NY
                    const li = document.createElement("li");
                    li.classList.add("panel-result");
                    li.tabIndex = 0;
                    li.setAttribute("data-result-id", index);
                    li.textContent = name;

                    fragment.appendChild(li);
                  });
                  // Empty the current list
                  listNode.innerHTML = "";
                  listNode.appendChild(fragment);
                })
                .catch(function(error) {
                  console.error("query failed: ", error);
                });
            }
          });
        

        // listen to click event on the zip code list
        listNode.addEventListener("click", onListClickHandler);

        function onListClickHandler(event) {
          const target = event.target;
          const resultId = target.getAttribute("data-result-id");

          // get the graphic corresponding to the clicked zip code
          const result =
            resultId && graphics && graphics[parseInt(resultId, 10)];

          if (result) {
            // open the popup at the centroid of zip code polygon
            // and set the popup's features which will populate popup content and title.

            view.goTo(result.geometry.extent.expand(2)).then(function() {
              view.popup.open({
                features: [result],
                location: result.geometry.centroid
              });
            });
          }
        }
      });

	//ПЕРЕМЕЩЕНИЕ РЕГИОНОВ ДОООО
		  //view.ui.add("titleDiv", "top-left");
		   view.ui.add(search, "top-right");
		  view.ui.add(homeBtn, "top-left");
		  view.ui.add(aboutExpand, "bottom-left");
		  view.ui.add(legend, "bottom-right");
          view.ui.add(seasonsExpand, "top-left");
		  }); 
      });
    </script>
  </head>
  <body>
  <div class="panel-container">
      <div class="panel-side esri-widget">
        <h3>Регионы Республики Башкортостан</h3>
        <ul id="nyc_graphics">
          <li>Загрузка&hellip;</li>
        </ul>
    </div>
    <div id="seasons-filter" class="esri-widget">
      <div class="season-item visible-season" form="производство молока">производство молока</div>
	  <div class="season-item visible-season" form="производство зерновых">производство зерновых</div>
	  <div class="season-item visible-season" form="переработка круп">переработка круп</div>
	  <div class="season-item visible-season" form="разведение КРС">разведение КРС</div>
	  <div class="season-item visible-season" form="производство мяса">производство мяса</div>
	  <div class="season-item visible-season" form="овощеводство">овощеводство</div>
	  <div class="season-item visible-season" form="производство круп">производсво круп</div>
	  <div class="season-item visible-season" form="свиноводство">свиноводство</div>			
      <div class="season-item visible-season" form="переработка молока">переработка молока</div>
      <div class="season-item visible-season" form="переработка масличных">переработка масличных</div>
    </div>
    <div id="viewDiv"></div>
    <div id="titleDiv" class="esri-widget">
      <div id="titleText">Крупнейшие сельхозхозяйственные предприятия РБ</div>
      <div>Актуальный перечень 2018 года</div>
    </div>
  </body>
</html>
