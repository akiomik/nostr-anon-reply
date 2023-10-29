<script lang="ts">
  import {
    nip19,
    validateEvent,
    getSignature,
    getEventHash,
    generatePrivateKey,
    getPublicKey,
    SimplePool,
  } from 'nostr-tools';

  let noteId = '';
  let hashtag = '#makeitquote';
  let status = '';

  const relays = ['wss://yabu.me', 'wss://nos.lol', 'wss://relay-jp.nostr.wirednet.jp'];

  const handleSumit = () => {
    status = '';

    const pool = new SimplePool();

    const sk = generatePrivateKey();
    console.debug('sk:', sk);

    const pk = getPublicKey(sk);
    console.debug('pk:', pk);

    const { data: etag } = nip19.decode(noteId);

    const ev = {
      kind: 1,
      created_at: Math.floor(Date.now() / 1000),
      tags: [
        ['e', etag],
        ['t', hashtag.substring(1)],
      ],
      content: `${hashtag}`,
      pubkey: pk,
    };
    ev.id = getEventHash(ev);
    ev.sig = getSignature(ev, sk);
    console.debug('ev:', ev, validateEvent(ev));

    Promise.all(pool.publish(relays, ev))
      .then(() => {
        console.debug('published!');
        status = 'published!';
      })
      .catch((e) => {
        console.error(e);
        status = 'error...';
      });
  };
</script>

<svelte:head>
  <title>Nostr Anonymous Reply</title>
  <meta name="description" content="Send nostr reply anonymously" />
</svelte:head>

<section>
  <h1 class="h1">Nostr Anonymous Reply</h1>

  <form on:submit={handleSumit}>
    <label class="label">
      <span>Note ID</span>
      <input
        bind:value={noteId}
        type="text"
        placeholder="note1..."
        minlength="63"
        maxlength="63"
        required
        class="input"
      />
    </label>

    <label class="label">
      <span>Hashtag</span>
      <input bind:value={hashtag} type="text" placeholder="#makeitquote" required class="input" />
    </label>

    <button type="submit" class="btn variant-filled mt-4">send</button>
  </form>

  {status}
</section>
