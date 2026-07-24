```dataviewjs
const pages = dv
    .pages('"Histoire/Chronologie/Périodes"')
    .array();

if (pages.length === 0) {
    dv.paragraph(
        "Aucune période trouvée. Vérifie le chemin du dossier et les propriétés."
    );
} else {
    /* =====================================================
       CONFIGURATION
       ===================================================== */

    const LARGEUR_LABEL = 150;

    // Compression de la préhistoire :
    // la période -20 000 → -4 000 occupe quatre fois moins de place.
    const COMPRESSION_ACTIVE = true;
    const COMPRESSION_DEBUT = -20000;
    const COMPRESSION_FIN = -4000;
    const FACTEUR_COMPRESSION = 0.25;

    const ZOOM_MIN = 0.5;
    const ZOOM_MAX = 40;
    const ZOOM_PAS = 0.25;

    /* =====================================================
       LARGEUR DE LA NOTE
       ===================================================== */

    const previewSizer = dv.container.closest(".markdown-preview-sizer");

    if (previewSizer) {
        previewSizer.style.maxWidth = "none";
        previewSizer.style.width = "100%";
    }

    const root = document.createElement("div");
    root.className = "frise-app";

    dv.container.appendChild(root);

    /* =====================================================
       CSS
       ===================================================== */

    const style = document.createElement("style");

    style.textContent = `
        .frise-app {
            width: 100%;
            max-width: none;
            margin: 0;
            color: var(--text-normal);
            font-family: var(--font-interface);
        }

        .frise-toolbar {
            display: flex;
            flex-wrap: wrap;
            align-items: end;
            gap: 12px;
            margin: 10px 0 12px;
            padding: 10px;
            border: 1px solid var(--background-modifier-border);
            border-radius: 9px;
            background: var(--background-secondary);
        }

        .frise-controle {
            display: flex;
            flex-direction: column;
            gap: 4px;
            min-width: 170px;
        }

        .frise-controle label {
            color: var(--text-muted);
            font-size: 12px;
            font-weight: 600;
        }

        .frise-controle input[type="text"],
        .frise-controle select {
            min-height: 34px;
            padding: 5px 9px;
            border: 1px solid var(--background-modifier-border);
            border-radius: 6px;
            background: var(--background-primary);
            color: var(--text-normal);
        }

        .frise-controle-zoom {
            min-width: 230px;
            flex: 1;
        }

        .frise-zoom-ligne {
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .frise-zoom-ligne input {
            flex: 1;
        }

        .frise-zoom-valeur {
            min-width: 58px;
            text-align: right;
            font-variant-numeric: tabular-nums;
        }

        .frise-bouton {
            min-height: 34px;
            padding: 5px 11px;
            border: 1px solid var(--background-modifier-border);
            border-radius: 6px;
            background: var(--interactive-normal);
            color: var(--text-normal);
            cursor: pointer;
        }

        .frise-bouton:hover {
            background: var(--interactive-hover);
        }

        .frise-layout {
            display: grid;
            grid-template-columns: minmax(0, 1fr) 290px;
            gap: 14px;
            align-items: start;
            width: 100%;
        }

        .frise-zone {
            min-width: 0;
        }

        .frise-scroll {
            width: 100%;
            overflow-x: auto;
            overflow-y: hidden;
            padding-bottom: 8px;
            overscroll-behavior-x: contain;
        }

        .frise-contenu {
            position: relative;
        }

        .frise-axe-ligne,
        .frise-ligne {
            display: grid;
            grid-template-columns: ${LARGEUR_LABEL}px 1fr;
            width: 100%;
        }

        .frise-axe-label {
            position: sticky;
            left: 0;
            z-index: 8;
            min-height: 36px;
            background: var(--background-primary);
            border-right: 1px solid var(--background-modifier-border);
        }

        .frise-axe {
            position: relative;
            height: 36px;
            border-bottom: 1px solid var(--background-modifier-border);
        }

        .frise-tick {
            position: absolute;
            top: 0;
            bottom: 0;
            width: 1px;
            background: var(--background-modifier-border);
        }

        .frise-tick-label {
            position: absolute;
            top: 2px;
            transform: translateX(-50%);
            color: var(--text-muted);
            font-size: 11px;
            white-space: nowrap;
        }

        .frise-ligne {
            min-height: 38px;
            margin: 1px 0;
        }

        .frise-ligne-label {
            position: sticky;
            left: 0;
            z-index: 7;
            display: flex;
            align-items: stretch;
            background: var(--background-primary);
            border-right: 1px solid var(--background-modifier-border);
        }

        .frise-ligne-bouton {
            display: flex;
            align-items: center;
            gap: 6px;
            width: 100%;
            min-height: 38px;
            padding: 4px 8px;
            border: none;
            border-radius: 0;
            background: transparent;
            color: var(--text-normal);
            cursor: pointer;
            font-weight: 600;
            text-align: left;
        }

        .frise-ligne-bouton:hover {
            background: var(--background-modifier-hover);
        }

        .frise-chevron {
            width: 12px;
            color: var(--text-muted);
            font-size: 10px;
        }

        .frise-piste {
            position: relative;
            height: 38px;
            border-bottom: 1px solid var(--background-modifier-border);
            background:
                repeating-linear-gradient(
                    to right,
                    transparent,
                    transparent calc(10% - 1px),
                    var(--background-modifier-border) 10%
                );
        }

        .frise-periode {
            position: absolute;
            top: 0;
            bottom: 0;
            display: flex;
            flex-direction: column;
            justify-content: center;
            height: 100%;
            min-width: 3px;
            padding: 3px 8px;
            overflow: hidden;
            border: none;
            border-left: 1px solid rgba(0, 0, 0, 0.2);
            border-right: 1px solid rgba(0, 0, 0, 0.2);
            border-radius: 4px;
            color: #191919;
            cursor: pointer;
            text-align: left;
        }

        .frise-periode:hover {
            z-index: 5;
            filter: brightness(1.1);
        }

        .frise-periode-selectionnee {
            z-index: 6;
            outline: 2px solid var(--interactive-accent);
            outline-offset: -2px;
        }

        .frise-date {
            position: absolute;
            top: 50%;
            z-index: 4;
            width: 16px;
            height: 16px;
            padding: 0;
            border: 2px solid var(--background-primary);
            border-radius: 50%;
            box-shadow: 0 0 0 1px rgba(0, 0, 0, 0.35);
            cursor: pointer;
            transform: translate(-50%, -50%);
        }

        .frise-date:hover {
            z-index: 7;
            filter: brightness(1.15);
            transform: translate(-50%, -50%) scale(1.18);
        }

        .frise-date.frise-periode-selectionnee {
            outline-offset: 2px;
        }

        .frise-periode-titre {
            display: flex;
            align-items: center;
            gap: 5px;
            overflow: hidden;
            font-size: 12px;
            font-weight: 700;
            text-overflow: ellipsis;
            white-space: nowrap;
        }

        .frise-periode-nom {
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
        }

        .frise-periode-dates {
            overflow: hidden;
            margin-top: 1px;
            font-size: 10px;
            opacity: 0.82;
            text-overflow: ellipsis;
            white-space: nowrap;
        }

        .frise-pastille-detail {
            display: inline-block;
            flex: 0 0 auto;
            width: 8px;
            height: 8px;
            border: 1px solid rgba(0, 0, 0, 0.45);
            border-radius: 50%;
            background: rgba(255, 255, 255, 0.9);
        }

        .frise-panneau {
            position: sticky;
            top: 16px;
            min-height: 210px;
            padding: 14px;
            border: 1px solid var(--background-modifier-border);
            border-radius: 9px;
            background: var(--background-secondary);
        }

        .frise-panneau-vide {
            color: var(--text-muted);
            font-size: 13px;
        }

        .frise-panneau h3 {
            margin: 0 0 10px;
            font-size: 18px;
        }

        .frise-panneau-dates {
            margin-bottom: 10px;
            font-weight: 600;
        }

        .frise-panneau-resume {
            margin: 12px 0;
            line-height: 1.5;
        }

        .frise-panneau-ligne {
            margin: 5px 0;
            color: var(--text-muted);
            font-size: 12px;
        }

        .frise-panneau-detaille {
            display: inline-flex;
            align-items: center;
            gap: 6px;
            margin-top: 7px;
            padding: 3px 7px;
            border-radius: 999px;
            background: var(--background-modifier-hover);
            font-size: 11px;
        }

        .frise-ouvrir-note {
            width: 100%;
            margin-top: 14px;
            padding: 8px 10px;
            border: none;
            border-radius: 7px;
            background: var(--interactive-accent);
            color: var(--text-on-accent);
            cursor: pointer;
            font-weight: 600;
        }

        .frise-informations {
            margin-top: 6px;
            color: var(--text-muted);
            font-size: 11px;
        }

        .frise-note-compression {
            margin-top: 3px;
            color: var(--text-muted);
            font-size: 10px;
        }

        @media (max-width: 900px) {
            .frise-layout {
                grid-template-columns: 1fr;
            }

            .frise-panneau {
                position: static;
            }
        }
    `;

    root.appendChild(style);

    /* =====================================================
       OUTILS
       ===================================================== */

    function texte(value, valeurParDefaut = "") {
        return value == null ? valeurParDefaut : String(value);
    }

    function nombre(value, valeurParDefaut = 9999) {
        const resultat = Number(value);
        return Number.isFinite(resultat)
            ? resultat
            : valeurParDefaut;
    }

    /*
       Accepte une année numérique (1789, -52), une date Obsidian
       (1789-07-14) ou un objet Date/Luxon fourni par Dataview.
    */
    function lireAnnee(value) {
        if (value == null) {
            return NaN;
        }

        if (
            typeof value === "object" &&
            Number.isFinite(Number(value.year))
        ) {
            return Number(value.year);
        }

        if (value instanceof Date && !Number.isNaN(value.getTime())) {
            return value.getFullYear();
        }

        const valeur = String(value).trim();

        if (/^-?\d+$/.test(valeur)) {
            return Number(valeur);
        }

        const dateIso = valeur.match(/^(-?\d{1,6})-\d{2}-\d{2}/);
        return dateIso ? Number(dateIso[1]) : NaN;
    }

    function normaliser(value) {
        return texte(value)
            .normalize("NFD")
            .replace(/[\u0300-\u036f]/g, "")
            .toLowerCase()
            .trim();
    }

    function estOui(value) {
        const valeur = normaliser(value);

        return [
            "oui",
            "yes",
            "true",
            "vrai",
            "1",
            "x"
        ].includes(valeur);
    }

    function afficherAnnee(annee) {
        const valeur = Math.round(Number(annee));

        if (valeur < 0) {
            return `${Math.abs(valeur)} av. J.-C.`;
        }

        if (valeur === 0) {
            return "début de notre ère";
        }

        return `${valeur} apr. J.-C.`;
    }

    function afficherDate(value) {
        if (value == null) {
            return "";
        }

        const valeur = String(value).trim();
        const dateIso = valeur.match(
            /^(\d{4,6})-(\d{2})-(\d{2})/
        );

        if (!dateIso) {
            return afficherAnnee(lireAnnee(value));
        }

        const [, annee, mois, jour] = dateIso;
        const date = new Date(Date.UTC(
            Number(annee),
            Number(mois) - 1,
            Number(jour)
        ));

        return new Intl.DateTimeFormat("fr-FR", {
            day: "numeric",
            month: "long",
            year: "numeric",
            timeZone: "UTC"
        }).format(date);
    }

    function valeursUniques(values) {
        return [...new Set(values)]
            .filter(Boolean)
            .sort((a, b) =>
                String(a).localeCompare(
                    String(b),
                    "fr",
                    { sensitivity: "base" }
                )
            );
    }

    /*
       Transformation non linéaire.

       Entre -20 000 et -4 000, les années sont comprimées.
       Avant et après, l'échelle reste linéaire.
    */
    function transformerAnnee(annee) {
        const y = Number(annee);

        if (!COMPRESSION_ACTIVE) {
            return y;
        }

        if (y <= COMPRESSION_DEBUT) {
            return y;
        }

        const longueurCompressee =
            (COMPRESSION_FIN - COMPRESSION_DEBUT) *
            FACTEUR_COMPRESSION;

        if (y < COMPRESSION_FIN) {
            return COMPRESSION_DEBUT +
                (y - COMPRESSION_DEBUT) *
                FACTEUR_COMPRESSION;
        }

        return COMPRESSION_DEBUT +
            longueurCompressee +
            (y - COMPRESSION_FIN);
    }

    function calculerPasIdeal(etendue, largeur) {
        const nombreReperes = Math.max(
            2,
            Math.floor(largeur / 150)
        );

        const pasBrut = etendue / nombreReperes;
        const puissance = Math.pow(
            10,
            Math.floor(Math.log10(Math.max(1, pasBrut)))
        );

        const fraction = pasBrut / puissance;

        if (fraction <= 1) {
            return puissance;
        }

        if (fraction <= 2) {
            return 2 * puissance;
        }

        if (fraction <= 5) {
            return 5 * puissance;
        }

        return 10 * puissance;
    }

    /* =====================================================
       DONNÉES
       ===================================================== */

    const periodes = pages
        .filter(page =>
            page.type === "periode" || page.type === "date"
        )
        .map(page => {
            const estDate = page.type === "date";
            const date = lireAnnee(page.date ?? page.annee);
            const debut = estDate ? date : lireAnnee(page.debut);
            const fin = estDate ? date : lireAnnee(page.fin);

            return {
                page,
                fichier: page.file,
                type: estDate ? "date" : "periode",
                dateOriginale: estDate
                    ? (page.date ?? page.annee)
                    : null,
                nom: texte(page.nom, page.file.name),
                civilisation: texte(
                    page.civilisation,
                    "Sans civilisation"
                ),
                debut,
                fin,
                couleur: texte(
                    page.couleur,
                    "var(--interactive-accent)"
                ),
                resume: texte(
                    page.resume,
                    "Aucun résumé n'a encore été renseigné."
                ),
                ordre: nombre(page.ordre),
                detaille: estOui(page.detaille)
            };
        })
        .filter(element =>
            Number.isFinite(element.debut) &&
            Number.isFinite(element.fin)
        );

    const civilisations = valeursUniques(
        periodes.map(periode => periode.civilisation)
    );

    /* =====================================================
       ÉTAT
       ===================================================== */

    const etat = {
        zoom: 1,
        recherche: "",
        civilisation: "Toutes",
        selection: null,
        lignesRepliees: new Set(),
        scrollLeft: 0
    };

    let scrollActuel = null;
    let blocsActuels = [];

    /* =====================================================
       BARRE D'OUTILS
       ===================================================== */

    const toolbar = document.createElement("div");
    toolbar.className = "frise-toolbar";
    root.appendChild(toolbar);

    const rechercheControle = document.createElement("div");
    rechercheControle.className = "frise-controle";

    const rechercheLabel = document.createElement("label");
    rechercheLabel.textContent = "Rechercher";

    const rechercheInput = document.createElement("input");
    rechercheInput.type = "text";
    rechercheInput.placeholder = "Nom, résumé, civilisation…";

    rechercheControle.appendChild(rechercheLabel);
    rechercheControle.appendChild(rechercheInput);
    toolbar.appendChild(rechercheControle);

    const civilisationControle = document.createElement("div");
    civilisationControle.className = "frise-controle";

    const civilisationLabel = document.createElement("label");
    civilisationLabel.textContent = "Civilisation";

    const civilisationSelect = document.createElement("select");

    civilisationSelect.appendChild(
        new Option("Toutes les civilisations", "Toutes")
    );

    for (const civilisation of civilisations) {
        civilisationSelect.appendChild(
            new Option(civilisation, civilisation)
        );
    }

    civilisationControle.appendChild(civilisationLabel);
    civilisationControle.appendChild(civilisationSelect);
    toolbar.appendChild(civilisationControle);

    const zoomControle = document.createElement("div");
    zoomControle.className =
        "frise-controle frise-controle-zoom";

    const zoomLabel = document.createElement("label");
    zoomLabel.textContent =
        "Zoom horizontal — Ctrl + molette sur la frise";

    const zoomLigne = document.createElement("div");
    zoomLigne.className = "frise-zoom-ligne";

    const zoomInput = document.createElement("input");
    zoomInput.type = "range";
    zoomInput.min = String(ZOOM_MIN);
    zoomInput.max = String(ZOOM_MAX);
    zoomInput.step = String(ZOOM_PAS);
    zoomInput.value = "1";

    const zoomValeur = document.createElement("span");
    zoomValeur.className = "frise-zoom-valeur";
    zoomValeur.textContent = "100 %";

    zoomLigne.appendChild(zoomInput);
    zoomLigne.appendChild(zoomValeur);
    zoomControle.appendChild(zoomLabel);
    zoomControle.appendChild(zoomLigne);
    toolbar.appendChild(zoomControle);

    const deplierBouton = document.createElement("button");
    deplierBouton.type = "button";
    deplierBouton.className = "frise-bouton";
    deplierBouton.textContent = "Tout déplier";
    toolbar.appendChild(deplierBouton);

    const resetBouton = document.createElement("button");
    resetBouton.type = "button";
    resetBouton.className = "frise-bouton";
    resetBouton.textContent = "Réinitialiser";
    toolbar.appendChild(resetBouton);

    /* =====================================================
       STRUCTURE
       ===================================================== */

    const layout = document.createElement("div");
    layout.className = "frise-layout";

    const zoneFrise = document.createElement("div");
    zoneFrise.className = "frise-zone";

    const panneau = document.createElement("aside");
    panneau.className = "frise-panneau";

    layout.appendChild(zoneFrise);
    layout.appendChild(panneau);
    root.appendChild(layout);

    /* =====================================================
       FILTRAGE ET DOMAINE
       ===================================================== */

    function obtenirPeriodesFiltrees() {
        const recherche = normaliser(etat.recherche);

        return periodes.filter(periode => {
            const civilisationValide =
                etat.civilisation === "Toutes" ||
                periode.civilisation === etat.civilisation;

            const texteRecherche = normaliser([
                periode.nom,
                periode.civilisation,
                periode.resume
            ].join(" "));

            const rechercheValide =
                recherche === "" ||
                texteRecherche.includes(recherche);

            return civilisationValide && rechercheValide;
        });
    }

    /*
       Lorsqu'une civilisation est choisie, le domaine temporel
       est calculé à partir de ses périodes uniquement.
    */
    function calculerDomaine(periodesFiltrees) {
        const periodesDuDomaine =
            etat.civilisation === "Toutes"
                ? periodes
                : periodes.filter(
                    periode =>
                        periode.civilisation ===
                        etat.civilisation
                );

        const source =
            periodesDuDomaine.length > 0
                ? periodesDuDomaine
                : periodesFiltrees;

        let debut = Math.min(
            ...source.map(periode => periode.debut)
        );

        let fin = Math.max(
            ...source.map(periode => periode.fin)
        );

        const etendue = Math.max(1, fin - debut);
        const marge = Math.max(10, etendue * 0.06);

        debut -= marge;
        fin += marge;

        return { debut, fin };
    }

    /* =====================================================
       PANNEAU
       ===================================================== */

    function afficherPanneau() {
        panneau.replaceChildren();

        if (!etat.selection) {
            const vide = document.createElement("div");
            vide.className = "frise-panneau-vide";
            vide.textContent =
                "Clique sur une période pour afficher ses informations.";

            panneau.appendChild(vide);
            return;
        }

        const periode = etat.selection;

        const titre = document.createElement("h3");
        titre.textContent = periode.nom;

        const dates = document.createElement("div");
        dates.className = "frise-panneau-dates";
        dates.textContent = periode.type === "date"
            ? afficherDate(periode.dateOriginale)
            : `${afficherAnnee(periode.debut)} — ` +
              `${afficherAnnee(periode.fin)}`;

        const civilisation = document.createElement("div");
        civilisation.className = "frise-panneau-ligne";
        civilisation.textContent =
            `Civilisation : ${periode.civilisation}`;

        const resume = document.createElement("div");
        resume.className = "frise-panneau-resume";
        resume.textContent = periode.resume;

        panneau.appendChild(titre);
        panneau.appendChild(dates);
        panneau.appendChild(civilisation);

        if (periode.type === "periode") {
            const duree = document.createElement("div");
            duree.className = "frise-panneau-ligne";
            duree.textContent =
                `Durée approximative : ` +
                `${Math.abs(periode.fin - periode.debut)} ans`;
            panneau.appendChild(duree);
        }

        panneau.appendChild(resume);

        if (periode.detaille) {
            const detaille = document.createElement("div");
            detaille.className = "frise-panneau-detaille";

            const pastille = document.createElement("span");
            pastille.className = "frise-pastille-detail";

            const texteDetaille = document.createElement("span");
            texteDetaille.textContent = "Fiche détaillée";

            detaille.appendChild(pastille);
            detaille.appendChild(texteDetaille);
            panneau.appendChild(detaille);
        }

        const ouvrir = document.createElement("button");
        ouvrir.type = "button";
        ouvrir.className = "frise-ouvrir-note";
        ouvrir.textContent = "Ouvrir la fiche Obsidian";

        ouvrir.addEventListener("click", () => {
            app.workspace.openLinkText(
                periode.fichier.path,
                dv.current().file.path,
                false
            );
        });

        panneau.appendChild(ouvrir);
    }

    function actualiserSelectionVisuelle() {
        for (const element of blocsActuels) {
            element.bloc.classList.toggle(
                "frise-periode-selectionnee",
                etat.selection?.fichier.path ===
                    element.periode.fichier.path
            );
        }
    }

    /* =====================================================
       POSITION ET ZOOM
       ===================================================== */

    function sauvegarderScroll() {
        if (scrollActuel) {
            etat.scrollLeft = scrollActuel.scrollLeft;
        }
    }

    function changerZoom(nouveauZoom, ancrageClientX = null) {
        if (!scrollActuel) {
            etat.zoom = nouveauZoom;
            afficherFrise();
            return;
        }

        const ancienZoom = etat.zoom;
        const ancienScroll = scrollActuel.scrollLeft;
        const ancienneLargeur =
            Number(scrollActuel.dataset.largeurChronologie) || 1;

        const rect = scrollActuel.getBoundingClientRect();

        const xDansScroll =
            ancrageClientX == null
                ? scrollActuel.clientWidth / 2
                : ancrageClientX - rect.left;

        const xChronologie =
            Math.max(0, xDansScroll - LARGEUR_LABEL);

        const positionObservee =
            ancienScroll + xChronologie;

        etat.zoom = Math.min(
            ZOOM_MAX,
            Math.max(ZOOM_MIN, nouveauZoom)
        );

        zoomInput.value = String(etat.zoom);
        zoomValeur.textContent =
            `${Math.round(etat.zoom * 100)} %`;

        afficherFrise({
            apresAffichage: nouveauScroll => {
                const nouvelleLargeur =
                    Number(
                        nouveauScroll.dataset.largeurChronologie
                    ) || 1;

                const rapport =
                    nouvelleLargeur / ancienneLargeur;

                nouveauScroll.scrollLeft =
                    positionObservee * rapport -
                    xChronologie;

                etat.scrollLeft =
                    nouveauScroll.scrollLeft;
            }
        });
    }

    /* =====================================================
       RENDU
       ===================================================== */

    function afficherFrise(options = {}) {
        sauvegarderScroll();

        zoneFrise.replaceChildren();
        blocsActuels = [];

        const periodesFiltrees = obtenirPeriodesFiltrees();

        if (periodesFiltrees.length === 0) {
            const vide = document.createElement("div");
            vide.textContent =
                "Aucune période ne correspond aux filtres.";

            zoneFrise.appendChild(vide);
            return;
        }

        const domaine = calculerDomaine(periodesFiltrees);

        const debutTransforme =
            transformerAnnee(domaine.debut);

        const finTransforme =
            transformerAnnee(domaine.fin);

        const etendueTransformee = Math.max(
            1,
            finTransforme - debutTransforme
        );

        const groupes = new Map();

        for (const periode of periodesFiltrees) {
            if (!groupes.has(periode.civilisation)) {
                groupes.set(periode.civilisation, []);
            }

            groupes
                .get(periode.civilisation)
                .push(periode);
        }

        const groupesTries = [...groupes.entries()]
            .map(([nom, elements]) => ({
                nom,
                elements,
                ordre: Math.min(
                    ...elements.map(element => element.ordre)
                )
            }))
            .sort((a, b) => {
                if (a.ordre !== b.ordre) {
                    return a.ordre - b.ordre;
                }

                return a.nom.localeCompare(
                    b.nom,
                    "fr",
                    { sensitivity: "base" }
                );
            });

        const largeurVisible = Math.max(
            zoneFrise.clientWidth - LARGEUR_LABEL,
            700
        );

        const largeurChronologie = Math.max(
            largeurVisible,
            largeurVisible * etat.zoom
        );

        const largeurTotale =
            LARGEUR_LABEL + largeurChronologie;

        const scroll = document.createElement("div");
        scroll.className = "frise-scroll";
        scroll.dataset.largeurChronologie =
            String(largeurChronologie);

        const contenu = document.createElement("div");
        contenu.className = "frise-contenu";
        contenu.style.width = `${largeurTotale}px`;

        scroll.appendChild(contenu);
        zoneFrise.appendChild(scroll);

        scrollActuel = scroll;

        /*
           Axe : les repères restent calculés en années réelles,
           mais leur position utilise l'échelle transformée.
        */
        const axeLigne = document.createElement("div");
        axeLigne.className = "frise-axe-ligne";

        const axeLabel = document.createElement("div");
        axeLabel.className = "frise-axe-label";

        const axe = document.createElement("div");
        axe.className = "frise-axe";

        const etendueReelle =
            domaine.fin - domaine.debut;

        const pas = calculerPasIdeal(
            etendueReelle,
            largeurChronologie
        );

        const premierRepere =
            Math.ceil(domaine.debut / pas) * pas;

        for (
            let annee = premierRepere;
            annee <= domaine.fin;
            annee += pas
        ) {
            const position =
                (
                    (
                        transformerAnnee(annee) -
                        debutTransforme
                    ) /
                    etendueTransformee
                ) * 100;

            if (position < 0 || position > 100) {
                continue;
            }

            const tick = document.createElement("div");
            tick.className = "frise-tick";
            tick.style.left = `${position}%`;

            const label = document.createElement("span");
            label.className = "frise-tick-label";
            label.style.left = `${position}%`;
            label.textContent = afficherAnnee(annee);

            axe.appendChild(tick);
            axe.appendChild(label);
        }

        axeLigne.appendChild(axeLabel);
        axeLigne.appendChild(axe);
        contenu.appendChild(axeLigne);

        for (const groupe of groupesTries) {
            const estReplie =
                etat.lignesRepliees.has(groupe.nom);

            const ligne = document.createElement("div");
            ligne.className = "frise-ligne";

            const labelZone = document.createElement("div");
            labelZone.className = "frise-ligne-label";

            const ligneBouton = document.createElement("button");
            ligneBouton.type = "button";
            ligneBouton.className = "frise-ligne-bouton";

            const chevron = document.createElement("span");
            chevron.className = "frise-chevron";
            chevron.textContent = estReplie ? "▶" : "▼";

            const nom = document.createElement("span");
            nom.textContent = groupe.nom;

            ligneBouton.appendChild(chevron);
            ligneBouton.appendChild(nom);
            labelZone.appendChild(ligneBouton);

            const piste = document.createElement("div");
            piste.className = "frise-piste";

            if (!estReplie) {
                groupe.elements
                    .sort((a, b) => a.debut - b.debut)
                    .forEach(periode => {
                        const debutPeriode =
                            transformerAnnee(periode.debut);

                        const finPeriode =
                            transformerAnnee(periode.fin);

                        const position =
                            (
                                (
                                    debutPeriode -
                                    debutTransforme
                                ) /
                                etendueTransformee
                            ) * 100;

                        const largeur =
                            (
                                (
                                    finPeriode -
                                    debutPeriode
                                ) /
                                etendueTransformee
                            ) * 100;

                        const estDate = periode.type === "date";
                        const bloc = document.createElement("button");

                        bloc.type = "button";
                        bloc.className = estDate
                            ? "frise-date"
                            : "frise-periode";
                        bloc.style.left = `${position}%`;

                        if (!estDate) {
                            bloc.style.width =
                                `${Math.max(largeur, 0.12)}%`;
                        }

                        bloc.style.background = periode.couleur;

                        bloc.title = estDate
                            ? `${periode.nom}\n` +
                              `${afficherDate(periode.dateOriginale)}\n\n` +
                              periode.resume
                            : `${periode.nom}\n` +
                              `${afficherAnnee(periode.debut)} – ` +
                              `${afficherAnnee(periode.fin)}\n\n` +
                              periode.resume;

                        if (!estDate) {
                            const titre =
                                document.createElement("span");

                            titre.className =
                                "frise-periode-titre";

                            const nomPeriode =
                                document.createElement("span");

                            nomPeriode.className =
                                "frise-periode-nom";

                            nomPeriode.textContent = periode.nom;

                            titre.appendChild(nomPeriode);

                            if (periode.detaille) {
                                const pastille =
                                    document.createElement("span");

                                pastille.className =
                                    "frise-pastille-detail";

                                pastille.title = "Fiche détaillée";
                                titre.appendChild(pastille);
                            }

                            const dates =
                                document.createElement("span");

                            dates.className =
                                "frise-periode-dates";

                            dates.textContent =
                                `${afficherAnnee(periode.debut)} – ` +
                                `${afficherAnnee(periode.fin)}`;

                            bloc.appendChild(titre);
                            bloc.appendChild(dates);
                        }

                        bloc.addEventListener("click", () => {
                            /*
                               On ne reconstruit pas la frise :
                               la position horizontale reste inchangée.
                            */
                            etat.selection = periode;
                            afficherPanneau();
                            actualiserSelectionVisuelle();
                        });

                        blocsActuels.push({
                            bloc,
                            periode
                        });

                        piste.appendChild(bloc);
                    });
            }

            ligneBouton.addEventListener("click", () => {
                sauvegarderScroll();

                if (etat.lignesRepliees.has(groupe.nom)) {
                    etat.lignesRepliees.delete(groupe.nom);
                } else {
                    etat.lignesRepliees.add(groupe.nom);
                }

                afficherFrise();
            });

            ligne.appendChild(labelZone);
            ligne.appendChild(piste);
            contenu.appendChild(ligne);
        }

        actualiserSelectionVisuelle();

        const informations = document.createElement("div");
        informations.className = "frise-informations";

        informations.textContent =
            `${periodesFiltrees.length} élément(s) affiché(s) — ` +
            `${afficherAnnee(domaine.debut)} à ` +
            `${afficherAnnee(domaine.fin)}.`;

        zoneFrise.appendChild(informations);

        if (
            COMPRESSION_ACTIVE &&
            domaine.debut < COMPRESSION_FIN
        ) {
            const compression =
                document.createElement("div");

            compression.className =
                "frise-note-compression";

            compression.textContent =
                `Échelle comprimée entre ` +
                `${afficherAnnee(COMPRESSION_DEBUT)} et ` +
                `${afficherAnnee(COMPRESSION_FIN)}.`;

            zoneFrise.appendChild(compression);
        }

        /*
           Ctrl + molette :
           le point situé sous la souris reste approximativement
           à la même position pendant le zoom.
        */
        scroll.addEventListener(
            "wheel",
            event => {
                if (!event.ctrlKey) {
                    return;
                }

                event.preventDefault();

                const direction =
                    event.deltaY < 0 ? 1 : -1;

                changerZoom(
                    etat.zoom + direction * ZOOM_PAS,
                    event.clientX
                );
            },
            { passive: false }
        );

        requestAnimationFrame(() => {
            if (typeof options.apresAffichage === "function") {
                options.apresAffichage(scroll);
            } else {
                scroll.scrollLeft = etat.scrollLeft;
            }
        });
    }

    /* =====================================================
       ÉVÉNEMENTS
       ===================================================== */

    let minuterieRecherche;

    rechercheInput.addEventListener("input", event => {
        clearTimeout(minuterieRecherche);

        minuterieRecherche = setTimeout(() => {
            etat.recherche = event.target.value;
            afficherFrise();
        }, 150);
    });

    civilisationSelect.addEventListener("change", event => {
        etat.civilisation = event.target.value;

        /*
           Une civilisation sélectionnée repart à 100 %,
           puis la frise recalcule automatiquement son domaine.
        */
        etat.zoom = 1;
        etat.scrollLeft = 0;

        zoomInput.value = "1";
        zoomValeur.textContent = "100 %";

        afficherFrise({
            apresAffichage: scroll => {
                scroll.scrollLeft = 0;
            }
        });
    });

    zoomInput.addEventListener("input", event => {
        changerZoom(Number(event.target.value));
    });

    deplierBouton.addEventListener("click", () => {
        etat.lignesRepliees.clear();
        afficherFrise();
    });

    resetBouton.addEventListener("click", () => {
        etat.zoom = 1;
        etat.recherche = "";
        etat.civilisation = "Toutes";
        etat.selection = null;
        etat.scrollLeft = 0;
        etat.lignesRepliees.clear();

        rechercheInput.value = "";
        civilisationSelect.value = "Toutes";
        zoomInput.value = "1";
        zoomValeur.textContent = "100 %";

        afficherPanneau();

        afficherFrise({
            apresAffichage: scroll => {
                scroll.scrollLeft = 0;
            }
        });
    });

    /* =====================================================
       PREMIER AFFICHAGE
       ===================================================== */

    afficherPanneau();
    afficherFrise();
}
```
