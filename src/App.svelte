<script>
  import { onMount } from 'svelte';
  // @ts-ignore
  import { openDB } from 'idb'

  let pages = [];
  let currentPageIndex = 0;
  let title = '';
  let note = '';
  let db;

  onMount(() => {
    openDB('notesApp', 1, {
      upgrade(db) {
        db.createObjectStore('pages', { keyPath: 'id', autoIncrement: true });
        db.createObjectStore('notes');
      },
    }).then(database => {
      db = database;
      return db.getAll('pages');
    }).then(savedPages => {
      if (savedPages.length > 0) {
        pages = savedPages.map(page => page.title);
        title = pages[currentPageIndex];
        return db.get('notes', title);
      } else {
        addPage();
      }
    }).then(savedNote => {
      if (savedNote !== undefined) {
        note = savedNote;
      }
    });
  });

  function saveNote() {
    const storedPageName = pages[currentPageIndex];
    if (storedPageName !== title) {
      db.delete('notes', storedPageName).then(() => {
        pages[currentPageIndex] = title;
        return db.put('pages', { id: currentPageIndex + 1, title });
      }).then(() => {
        return db.put('notes', note, title);
      });
    } else {
      db.put('notes', note, title);
    }
  }

  function addPage() {
    const newPageTitle = "New Page";
    pages.push(newPageTitle);
    db.put('pages', { id: pages.length, title: newPageTitle }).then(() => {
      selectPage(pages.length - 1);
    });
  }

  function selectPage(index) {
    currentPageIndex = index;
    title = pages[currentPageIndex];
    db.get('notes', title).then(savedNote => {
      note = savedNote;
    });
  }

  function deletePage(index) {
  const pageToDelete = pages[index];
  db.delete('notes', pageToDelete).then(() => {
    return db.delete('pages', index);
  }).then(() => {
    pages.splice(index, 1);
    if (pages.length === 0) {
      title = '';
      note = '';
      currentPageIndex = 0;
    } else {
      currentPageIndex = index > 0 ? index - 1 : 0;
      selectPage(currentPageIndex);
    }
  });
}
</script>


<aside class="fixed top-0 z-40 w-60 h-screen">
  <div style="background-color: #F7F7F7" class="overflow-y-auto py-5 px-3 h-full border-r border-gray-200">
    {#if pages.length > 0}
      <ul class="space-y-2">
        {#each pages as page, index}
          <li class="flex items-center justify-between py-2 px-3">
            <button on:click={() => selectPage(index)} style={`background-color: ${index == currentPageIndex? '#E5E5E5' : 'transparent'}`} class="w-full text-gray-900 rounded-lg">
              {page}
            </button>
            <button on:click={() => deletePage(index)} style="color: #FFC080" class="ml-2 hover:text-red-700 transition duration-300 ease-in-out">
              <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm3-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z" clip-rule="evenodd" />
              </svg>
            </button>
          </li>
        {/each}
      </ul>
    {/if}
    <div class="text-center mt-4">
      <button on:click={addPage} style="background-color: #2F2F2F; color: #FFFFFF" class="font-medium py-2 px-4 rounded-lg hover:bg-gray-900">
        + Add Page
      </button>
    </div>
  </div>
</aside>

<main class="p-4 ml-60 h-auto">
  <h1 style="font-size: 36px; font-weight: bold; margin-bottom: 16px" class="text-center">Quick Notes</h1>
  {#if pages.length > 0}
    <div class="grid grid-cols-2 items-center mb-3">
      <h1 style="font-size: 24px; font-weight: bold" contenteditable bind:textContent={title}>{title || "New Page"}</h1>
      <button on:click={saveNote} style="background-color: #2F2F2F; color: #FFFFFF" class="ml-auto font-medium py-2 px-4 rounded-lg hover:bg-gray-900">
        Save
      </button>
    </div>
    <hr style="margin-bottom: 16px"/>
    <textarea style="width: 100%; background-color: #F7F7F7; border: 1px solid #E5E5E5; padding: 10px; border-radius: 10px" bind:value={note}></textarea>
  {/if}
</main>