---
title: Méthodologie
---

<div class="content">

<h5>Méthodologie</h5>

<p>Afin de retracer l'évolution hydrologique de Lausanne, notre projet s'appuie sur une approche hybride croisant cartographie numérique (SIG), science des données et recherche historique. Ce travail a été structuré en quatre étapes clés.</p>

<h4>1. Sélection des sources et ancrage chronologique</h4>
<p>Nous avons sélectionné des sources cartographiques produites sur trois siècles pour identifier les moments de rupture du développement urbain : les cadastres de Melotte (1721) et de Berney (1831) pour l'époque pré-industrielle, le cadastre Rénové (1888) pour l'entrée dans l'ère industrielle, les plans de ville de 1937 et 1959 pour l'expansion moderne, et enfin les données de 2021 comme référentiel contemporain.</p>

<h4>2. Traitement spatial et vectorisation (QGIS)</h4>
<p>L'utilisation de ces documents a nécessité un travail de géoréférencement systématique sur le système de coordonnées suisse officiel (EPSG:2056). Nous avons ensuite procédé à une numérisation vectorielle rigoureuse : chaque segment de cours d'eau a été tracé et nommé en croisant les informations textuelles des plans avec les données historiques disponibles. Ce travail a également inclus le dessin de la rive du lac, traité par une double approche : une couche linéaire pour le trait de côte et une couche polygonale pour l'aire lacustre.</p>

<p>Pour garantir la précision des mesures de remblais, les polygones du lac ont été créés avec une contrainte géométrique stricte : leurs contours "terrestres" ont été maintenus identiques d'une période à l'autre, seule la ligne de rive divergeant selon les tracés historiques. Afin d'assurer une comparaison statistique fiable, l'ensemble des couches a été rogné selon une emprise spatiale fixe, garantissant que l'aire d'étude demeure constante à travers les siècles.</p>

<p>Pour les époques disposant de données polygonales détaillées (Melotte, Berney, Rénové et période contemporaine), nous avons également analysé la mutation de l'occupation du sol. En isolant les segments de rivières qui disparaissaient entre deux époques, nous avons réalisé des intersections spatiales avec les couches de polygones de la période suivante. Cette méthode a permis d'identifier précisément la nature des infrastructures ou des usages (bâti, axes de circulation, jardins) ayant succédé au réseau hydrographique en surface lors de son effacement.</p>

<h4>3. Analyse quantitative et statistique (Python)</h4>
<p>Une fois la base de données spatiales constituée, nous avons développé des scripts Python pour extraire et traiter des mesures précises, assurant une comparaison normalisée entre les époques :</p>
<ul>
<li><strong>Analyse des réseaux fluviaux :</strong> Nous avons calculé la longueur totale des réseaux pour chaque époque, segmentée par nom de rivière. Ce traitement statistique a permis de quantifier précisément le taux d'effacement ou de déviation pour chaque cours d'eau spécifique à travers les siècles.</li>
<li><strong>Analyse des successions d'occupation du sol :</strong> À partir des intersections spatiales réalisées, nous avons quantifié les types de surfaces ayant remplacé les cours d'eau disparus. Ces scripts ont permis d'extraire la répartition proportionnelle (en surfaces) des infrastructures (bâti, routes) et des espaces verts ayant succédé au réseau hydrographique.</li>
<li><strong>Mesure des déplacements par zones tampons :</strong> Pour quantifier l'évolution du tracé des rivières, nous avons utilisé une analyse par "buffers". Cette méthode a permis de mesurer l'écart spatial entre les anciens lits et les nouveaux.</li>
<li><strong>Évolution du littoral et des remblais :</strong> Pour le lac, nous avons calculé les variations de surface des polygones de rive d'une période à l'autre. En croisant ces données avec le trait de côte linéaire, nous avons pu déterminer le volume de surface gagnée sur l'eau et la distance moyenne de recul du lac par rapport à son état de 1721.</li>
</ul>

<h4>4. Contextualisation et recherche documentaire</h4>
<p>Cette étape finale consiste à croiser les résultats quantitatifs avec une recherche documentaire s'appuyant sur la littérature académique spécialisée et la plateforme <em>impresso</em>. Le recours à la presse historique permet de corréler systématiquement les données géométriques aux contextes sociopolitiques et aux décisions d'aménagement contemporains des relevés cartographiques. Cette comparaisons des sources permet d'associer un cadre explicatif aux évolutions mesurées, transformant ainsi les observations spatiales et statistiques brutes en une analyse historique cohérente du territoire lausannois.</p>
</div>