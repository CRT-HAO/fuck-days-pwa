<script lang="ts">
  import "./app.css";
  import "./styles.css";

  import * as AlertDialog from "$lib/components/ui/alert-dialog";
  import { Button } from "$lib/components/ui/button";
  import { toast } from "svelte-sonner";
  import { localStorageWritable } from "@babichjacob/svelte-localstorage";
  import ReloadPromt from "$lib/ReloadPromt.svelte";
  import { Toaster } from "$lib/components/ui/sonner";
  import { ModeWatcher } from "mode-watcher";
  import { ScrollArea } from "$lib/components/ui/scroll-area";
  import { Separator } from "$lib/components/ui/separator";
  import { onMount } from "svelte";
  import stuck from "@zirkeldesign/tailwindcss-stuck-variant/src/observer";

  const storeDates = localStorageWritable<number[]>("dates", []);

  let card: HTMLElement | undefined = undefined;

  let innerHeight = 0;
  $: marginTop = innerHeight / 2 - (card !== undefined ? card.clientHeight : 0);

  function handleFuck() {
    const now = Date.now();
    $storeDates = [...$storeDates, now];
    toast("You fucked again 😔");
  }

  function handleDeleteDate(index: number) {
    $storeDates = $storeDates.filter((_, i) => i !== index);
    toast("The date has been deleted");
  }

  function toDays(millis: number) {
    return Math.floor(millis / 1000 / (3600 * 24));
  }

  $: lastDates = $storeDates.length > 0 ? $storeDates.at(-1)! : Date.now();

  // This calculation method is counted as a new day after 00:00
  $: days = toDays(Date.now()) - toDays(lastDates);

  $: dates = $storeDates
    .map((v) => {
      const d = new Date(v);
      return d.toLocaleString();
    })
    .reverse();

  onMount(() => {
    stuck();
  });
</script>

<svelte:head>
  <title>Fuck Days</title>
  <meta name="description" content="Fuck sDays counter" />
</svelte:head>

<svelte:window bind:innerHeight />

<div class="app">
  <main style="margin-top: {marginTop}px">
    <section class="flex flex-nowrap flex-col justify-start items-center gap-4">
      <div
        bind:this={card}
        class="flex flex-col w-full bg-background stuck:shadow-xl transition-shadow duration-200 px-4 py-6 rounded-b-xl gap-2 text-center sticky top-0 z-20"
      >
        <div>
          <h1 class="text-primary text-6xl">
            {days} <small class="text-4xl">days</small>
          </h1>
          <span>
            since {new Date(lastDates).toLocaleString()}
          </span>
        </div>

        <Button class="w-full" on:click={() => handleFuck()}>Fuck</Button>
      </div>

      {#if dates.length > 0}
        <ScrollArea class="w-full p-4">
          <div class="p-4">
            <h4 class="mb-4 text-sm font-medium leading-none">Fuck Dates</h4>
            {#each dates as date, i}
              <div
                class="flex flex-nowrap justify-between items-center text-sm"
              >
                <div>{date}</div>
                <div>
                  <AlertDialog.Root>
                    <AlertDialog.Trigger asChild let:builder>
                      <Button variant="link" class="h-6" builders={[builder]}
                        >Delete</Button
                      >
                    </AlertDialog.Trigger>
                    <AlertDialog.Content>
                      <AlertDialog.Header>
                        <AlertDialog.Title
                          >Do you really want to delete it?</AlertDialog.Title
                        >
                        <AlertDialog.Description>
                          This action cannot be undone.
                        </AlertDialog.Description>
                      </AlertDialog.Header>
                      <AlertDialog.Footer>
                        <AlertDialog.Cancel>Cancel</AlertDialog.Cancel>
                        <AlertDialog.Action
                          on:click={() =>
                            handleDeleteDate(dates.length - 1 - i)}
                          >Delete</AlertDialog.Action
                        >
                      </AlertDialog.Footer>
                    </AlertDialog.Content>
                  </AlertDialog.Root>
                </div>
              </div>
              <Separator class="my-2" />
            {/each}
          </div>
        </ScrollArea>
      {/if}
    </section>

    <ModeWatcher />
    <Toaster />
    <ReloadPromt />
  </main>
</div>

<style>
  h1 {
    width: 100%;
  }
</style>
