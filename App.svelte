<script>
  // Sat, 28 Sep 2019 21:38:13 +0000
  const formatDate = dateString => {
    const parsed = new Date(dateString);
    return `${parsed.toLocaleDateString()} ${parsed.toLocaleTimeString()}`;
  }

  const getHost = url => new URL(url).hostname;

  const getNews = async param => {
    if (param) {
      const res = await fetch(`https://rss-worker.snackylab.workers.dev/${param}`);
      const data = await res.json();
      return memozie(param, true, data.items);
    } else {
      throw new Error();
    }
  };

  const param = window.location.search;

  const encode = obj => JSON.stringify(obj);
  const decode = str => JSON.parse(str);

  const memozie = (param, write, data) => {
    if (window.localStorage && btoa) {
      const key = btoa(param);
      if (!write) { // read mode
        const saved = window.localStorage.getItem(key);
        if (saved) {
          const found = decode(saved);
          if (found) {
            if (Date.now() - found.time <= 5 * 60 * 1000) { // Cache for 5 min
              console.log("Found cached data.");
              return found.data;
            } else {
              console.log("Cache expired.");
            }
          }
        }
      } else { // write mode
        window.localStorage.setItem(key, encode({
          data: data,
          time: Date.now()
        }));
        return data;
      }
    }
    return getNews(param);
  };

  let loader = memozie(param);

</script>

{#await loader}
<p>Loading...</p>
{:then feeds}
<div class="container">
  {#each feeds as item, i}
  <a class="news-item" target="_blank" rel="noreferrer noopener" href={item.link}>
    <div class="item-title">{item.title}</div>
    <div class="date-string">{getHost(item.link)} | {formatDate(item.date)}</div>
  </a>
  {/each}
</div>
{:catch error}
<pre>
Hello! Welcome to QuickRSS!

In case you do not know, QuickRSS is an easy to use Web application
help you quickly read any RSS feeds right on the browser.

To use, simply put the RSS feed URL in the query param, for example:

    https://quickrss.now.sh/?url=https://news.ycombinator.com/rss

You can even put multiple RSS feeds, each one separated by a comma:

    https://quickrss.now.sh/?url=feed-1,feed-2,feed-3

To avoid heavy load on the server, we saved the RSS data in your
browser's localStorage for 5 minutes.
</pre>
{/await}

<style>
* { font-family: Trebuchet MS, Lucida Grande, Lucida Sans Unicode, Lucida Sans, Tahoma, sans-serif; }
pre { font-family: monospace; }
body { font-size: 14px; line-height: 20px; }
.container {
}

a.news-item {
  display: block;
  padding: 15px 10px;
  border-bottom: 1px solid #eee;
  text-decoration: none;
  color: #000;
}

.item-title {
  color: #2D6DF8;
}

.date-string {
  color: #888;
  margin-top: 5px;
  font-size: 0.8em;
}

a.news-item:hover .item-title {
  opacity: 0.5;
}
a.news-item:visited .item-title {
  color: #eee;
}
a.news-item:visited .date-string {
  color: #eee;
}

</style>
