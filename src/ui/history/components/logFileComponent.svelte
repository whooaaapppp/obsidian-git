<script lang="ts">
    import { setIcon, TFile } from "obsidian";
    import { DIFF_VIEW_CONFIG } from "src/constants";
    import { DiffFile } from "src/types";
    import { getDisplayPath, getNewLeaf } from "src/utils";
    import HistoryView from "../historyView";

    export let diff: DiffFile;
    export let view: HistoryView;
    let buttons: HTMLElement[] = [];

    $: side = (view.leaf.getRoot() as any).side == "left" ? "right" : "left";

    window.setTimeout(
        () => buttons.forEach((b) => setIcon(b, b.getAttr("data-icon")!)),
        0
    );

    function open(event: MouseEvent) {
        const file = view.app.vault.getAbstractFileByPath(diff.vault_path);

        if (file instanceof TFile) {
            getNewLeaf(event)?.openFile(file);
        }
    }

    function showDiff(event: MouseEvent) {
        getNewLeaf(event)?.setViewState({
            type: DIFF_VIEW_CONFIG.type,
            active: true,
            state: {
                file: diff.path,
                staged: false,
                hash: diff.hash,
            },
        });
    }
</script>

<main
    on:click|stopPropagation={showDiff}
    on:auxclick|stopPropagation={showDiff}
    on:focus
    class="tree-item nav-file"
>
    <div
        class="tree-item-self is-clickable nav-file-title"
        class:is-active={view.plugin.lastDiffViewState?.file ==
            diff.vault_path && view.plugin.lastDiffViewState?.hash}
        data-path={diff.vault_path}
        aria-label-position={side}
        data-tooltip-position={side}
        aria-label={diff.vault_path}
    >
        <div class="tree-item-inner nav-file-title-content">
            {getDisplayPath(diff.vault_path)}
        </div>
        <div class="git-tools">
            <div class="buttons">
                {#if view.app.vault.getAbstractFileByPath(diff.vault_path)}
                    <div
                        data-icon="go-to-file"
                        aria-label="Open File"
                        bind:this={buttons[0]}
                        on:auxclick|stopPropagation={open}
                        on:click|stopPropagation={open}
                        class="clickable-icon"
                    />
                {/if}
            </div>
            <span class="type" data-type={diff.status}>{diff.status}</span>
        </div>
    </div>
</main>

<style lang="scss">
    main {
        .nav-file-title {
            align-items: center;
        }
    }
</style>
