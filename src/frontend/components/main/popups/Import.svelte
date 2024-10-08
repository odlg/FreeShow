<script lang="ts">
    import { tick } from "svelte"
    import { IMPORT } from "../../../../types/Channels"
    import { Popups } from "../../../../types/Main"
    import { convertText } from "../../../converters/txt"
    import { activePopup, alertMessage, dataPath, os } from "../../../stores"
    import { send } from "../../../utils/request"
    import T from "../../helpers/T.svelte"
    import Button from "../../inputs/Button.svelte"

    const freeshow_formats = [
        { name: "Show", extensions: ["show", "json"], id: "freeshow" },
        { name: "Project", extensions: ["project", "shows", "json"], id: "freeshow_project" }, // , "fsproject", "fsp"
        { name: "Template", extensions: ["fstemplate", "fst", "template", "json"], id: "freeshow_template" },
        { name: "Theme", extensions: ["fstheme", "theme", "json"], id: "freeshow_theme" },
        // { name: "Calendar", extensions: ["ics"], id: "calendar" }, // calendar drawer tab
        // { name: "Scripture", id: "scripture" }, // scripture drawer tab
    ]

    const text_formats: { popup?: Popups; [key: string]: any }[] = [
        { name: "Clipboard", id: "clipboard" },
        { name: "Text file", extensions: ["txt"], id: "txt" },
        { name: "ChordPro", extensions: ["cho", "crd", "chopro", "chordpro", "chord", "pro", "txt", "onsong"], id: "chordpro" },
        {
            name: "PowerPoint",
            extensions: ["ppt", "pptx"],
            id: "powerpoint",
            tutorial:
                "This will import the plain text as a show." +
                ($os === "linux" ? "" : " If you would like to use a PowerPoint/Keynote presentation with FreeShow, please drag and drop it into your project.") +
                " Or you can import directly as PDF or images if you don't need animations.",
        },
        { name: "Word", extensions: ["doc", "docx"], id: "word" },
        { name: "ProPresenter", extensions: ["pro4", "pro5", "pro6", "pro", "json"], id: "propresenter" },
        {
            name: "EasyWorship",
            extensions: ["db"],
            id: "easyworship",
            tutorial: "Import the SongsWords.db file from the Data folder - Often located in the Documents folder<br>Optionally select Songs.db to import title/metadata",
        },
        {
            name: "VideoPsalm",
            extensions: ["json"],
            id: "videopsalm",
            tutorial: "1. Find the .vpc or .json file(s) - Often located in Documents\\VideoPsalm\\Songbooks<br>2. If it's .vpc, add .zip to the end & extract<br>3. Import the .json file(s)",
        },
        { name: "OpenLP/OpenLyrics", extensions: ["xml", "sqlite"], id: "openlp" },
        { name: "OpenSong", extensions: [], id: "opensong" },
        { name: "SoftProjector", extensions: ["sps"], id: "softprojector" },
        { name: "Songbeamer", id: "songbeamer", popup: "songbeamer_import" },
    ]

    const media_formats = [
        { name: "PDF", extensions: ["pdf"], id: "pdf" },
        { name: "Lessons.church", extensions: ["json", "olp", "olf"], id: "lessons" },
    ]

    function displayTutorial(format: any) {
        if (!format.tutorial) {
            activePopup.set(null)
            return
        }

        alertMessage.set(format.tutorial)
        activePopup.set("alert")
    }

    function importFromClipboard() {
        navigator.clipboard
            .readText()
            .then((text) => {
                convertText({ text, noFormatting: true })
            })
            .catch((err) => {
                console.error("Failed to read clipboard contents: ", err)
            })
    }
</script>

<h3>FreeShow</h3>
<div style="margin-bottom: 20px;display: flex;">
    {#each freeshow_formats as format}
        <Button
            style="flex: 1;min-height: 50px;"
            on:click={() => {
                send(IMPORT, [format.id], { path: $dataPath, format })
                displayTutorial(format)
            }}
            center
        >
            <img style="height: 60px;" src="./import-logos/freeshow.webp" alt="FreeShow-logo" />
            <p style="margin-left: 5px;">{format.name}</p>
        </Button>
    {/each}
</div>

<h3><T id="settings.text_import" /></h3>
<div style="margin-bottom: 20px;">
    {#each text_formats as format}
        <Button
            style="width: 20%;flex-direction: column;min-height: 160px;"
            on:click={() => {
                if (format.popup) {
                    tick().then(() => {
                        if (format.popup) {
                            activePopup.set(format.popup)
                        }
                    })
                } else if (format.extensions) {
                    send(IMPORT, [format.id], { path: $dataPath, format })
                    displayTutorial(format)
                } else if (format.id === "clipboard") {
                    importFromClipboard()
                    activePopup.set(null)
                }
            }}
            bold={false}
            center
        >
            <img src="./import-logos/{format.id}.webp" alt="{format.id}-logo" />
            <p>{format.name}</p>
        </Button>
    {/each}
</div>

<h3><T id="settings.media_import" /></h3>
<div>
    {#each media_formats as format}
        <Button
            style="width: 20%;flex-direction: column;min-height: 160px;"
            on:click={() => {
                send(IMPORT, [format.id], { format })
                displayTutorial(format)
            }}
            bold={false}
            center
        >
            <img src="./import-logos/{format.id}.webp" alt="{format.id}-logo" />
            <p>{format.name}</p>
        </Button>
    {/each}
</div>

<style>
    div {
        display: flex;
        flex-wrap: wrap;
    }

    img {
        height: 100px;
        max-width: 100%;
        object-fit: contain;
        padding: 10px;
    }

    h3 {
        font-size: 1.2em;
        color: var(--text);
        margin-bottom: 5px;
    }
</style>
