<style scoped>
table,
th,
td {
  border: 1px solid;
}
em {
  color: yellow;
}
.highlight-0 {
  color: red;
  font: bold;
  font-weight: bolder;
}
.highlight-1 {
  color: green;
  font: bold;
  font-weight: bolder;
}
</style>
<template>
  <input type="text" placeholder="Keywords" v-model="query" /><button
    @click="refresh()"
  >
    検索
  </button>
  <br />
  <div class="books">
    <table>
      <tr>
        <th>Title</th>
        <th>Description</th>
        <th>Author</th>
        <th>Publisher</th>
        <th>PublishedDate</th>
        <th>Id</th>
      </tr>
      <tr v-for="book in books">
        <td>
          <Highlighter
            class="my-highlight"
            highlightClassName="highlight"
            :searchWords="query?.split(',') || []"
            :autoEscape="true"
            :textToHighlight="book.title"
            v-slot="items"
          >
            <div>
              <template v-for="{ chunk, text, attrs } in items">
                <span
                  v-if="chunk.highlight"
                  v-bind="attrs"
                  :key="attrs.key"
                  :class="['highlight-' + query.split(',').indexOf(text)]"
                  >{{ text }}</span
                >
                <template v-else
                  ><span :style="{ color: 'black' }">{{ text }}</span></template
                >
              </template>
            </div>
          </Highlighter>
        </td>
        <td>
          <Highlighter
            class="my-highlight"
            highlightClassName="highlight"
            :searchWords="query?.split(',') || []"
            :autoEscape="true"
            :textToHighlight="book.description"
            v-slot="items"
          >
            <template v-for="{ chunk, text, attrs } in items">
              <span
                v-if="chunk.highlight"
                v-bind="attrs"
                :key="attrs.key"
                :class="['highlight-' + query.split(',').indexOf(text)]"
                >{{ text }}</span
              >
              <template v-else
                ><span :style="{ color: 'black' }">{{ text }}</span></template
              >
            </template>
          </Highlighter>
        </td>
        <td>{{ book.authors }}</td>
        <td>{{ book.publisher }}</td>
        <td>{{ book.publishedDate }}</td>
        <td>{{ book.id }}</td>
      </tr>
    </table>
  </div>
</template>

<script setup>
import Highlighter from "vue-highlight-words";

const max = 200;
const query = ref("");

const substract = function (text) {
  console.log("text", text?.length);
  const length = text?.length || 0;
  if (length <= max) {
    return text;
  }
  const quries = query.value?.split(",");
  if (!quries) {
    return;
  }
  console.log("quries", quries);
  let foundIdx = -1;
  for (let i = 0; i < quries.length; i++) {
    const q = quries[i];
    foundIdx = text.indexOf(q);
    if (foundIdx !== -1) {
      break;
    }
  }
  console.log("foundIdx", foundIdx);
  if (foundIdx === -1) {
    return text.substring(0, max - 1) + "......";
  }
  if (length - foundIdx - 1 > foundIdx) {
    const start = foundIdx > max / 2 ? max / 2 : foundIdx - 1;
    console.log("start", start, length - foundIdx - 1 - foundIdx + start);

    return (
      text.substring(start, length - foundIdx - 1 - foundIdx + start) + "......"
    );
  } else {
    const end = length - foundIdx - 1 > max / 2 ? foundIdx + max / 2 : max - 1;
    console.log("end", foundIdx - max / 2 + (end - foundIdx), end);
    return (
      text.substring(foundIdx - max / 2 + (end - foundIdx), end) + "......"
    );
  }
};
const { data: books, refresh } = await useAsyncData("books", async () => {
  console.log("query", query.value);
  return query.value
    ? $fetch(
        `https://7hpagfdd28.execute-api.ap-northeast-2.amazonaws.com/dd-to-opensearch?search=${query.value}`
      )
        .then((res) => {
          const response = JSON.parse(res);
          console.log("response", response);
          return response?.body?.hits?.hits?.map((book) => {
            console.log("book", book);
            const mapped = {
              ...book._source,
              description:
                book.highlight?.description
                  ?.join("<br/>")
                  .replaceAll("<em>", "")
                  .replaceAll("</em>", "") ||
                substract(book._source?.description),
              id: book._id,
            };
            console.log("mapped", mapped);
            return mapped;
          });
        })
        .then((books) => {
          console.log("books", books);
          return books;
        })
    : Promise.resolve(null);
});
</script>
