<script lang="ts">
  import {
    nip19,
    validateEvent,
    getSignature,
    getEventHash,
    generatePrivateKey,
    getPublicKey,
    SimplePool,
    type UnsignedEvent,
    type Event,
  } from 'nostr-tools';

  let noteId = '';
  let content = '#makeitquote';
  let status = '';

  const relays = ['wss://yabu.me', 'wss://nos.lol', 'wss://relay-jp.nostr.wirednet.jp'];

  const handleSumit = () => {
    status = '';

    const pool = new SimplePool();

    const hashtags = (
      [
        content.match(/^(#[\w]+)/),
        ...content.matchAll(/\s(#[\w]+)/g),
      ] as (RegExpMatchArray | null)[]
    )
      .filter((a): a is RegExpMatchArray => a !== null)
      .map(([, match]) => match);
    console.debug('hashtags:', hashtags);

    const sk = generatePrivateKey();
    console.debug('sk:', sk);

    const pk = getPublicKey(sk);
    console.debug('pk:', pk);

    const { data: etag } = nip19.decode(noteId);
    // TODO: Improve type safety for nip19.decode result handling
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    const eventId = typeof etag === 'string' ? etag : (etag as any).id;

    const unsignedEvent: UnsignedEvent<1> = {
      kind: 1,
      created_at: Math.floor(Date.now() / 1000),
      tags: [
        ['e', eventId],
        ...hashtags.map((hashtag) => {
          return ['t', hashtag.substring(1)];
        }),
      ],
      content,
      pubkey: pk,
    };

    const ev: Event<1> = {
      ...unsignedEvent,
      id: getEventHash(unsignedEvent),
      sig: getSignature(unsignedEvent, sk),
    };
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

  <aside class="alert variant-ghost-warning mt-4">
    <div class="alert-message">
      <h3 class="h3">Warning</h3>
      <p>
        This tool send a reply with a random public key, but your IP may be logged in each relay.
      </p>
    </div>
  </aside>

  <form on:submit={handleSumit} class="mt-4">
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
      <span>Reply</span>
      <textarea
        bind:value={content}
        class="textarea"
        required
        rows="4"
        placeholder="#makeitquote"
      />
    </label>

    <button type="submit" class="btn variant-filled mt-4">Send</button>
  </form>

  {status}
</section>
