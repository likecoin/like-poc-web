<template>
  <md-image :md-src="ipfsUrl" />
</template>

<script>
import axios from 'axios';
import IPFS_HOST from '@/api/ipfsHost';

export default {
  name: 'MdIpfsImage',
  props: {
    ipfsSrc: String,
  },
  data() {
    return {
      ipfsHost: 'https://ipfs.io/ipfs',
    };
  },
  computed: {
    ipfsUrl() {
      return `${this.ipfsHost}/${this.ipfsSrc}`;
    },
  },
  methods: {
    pingForIPFSHost(hash) {
      Promise.race([
        axios.head(`https://ipfs.io/ipfs/${hash}`),
        new Promise(resolve => setTimeout(resolve, 1000))
        .then(() => axios.head(`${IPFS_HOST}/${hash}`)),
      ])
      .then((res) => {
        if (res.status < 300) {
          this.ipfsHost = res.request.responseURL.replace(`/${hash}`, '');
        }
      })
      .catch((err) => {
        console.log(`Err: ${err}`);
      });
    },
  },
  watch: {
    ipfsSrc() {
      this.pingForIPFSHost(this.ipfsSrc);
    },
  },
  mounted() {
    this.pingForIPFSHost(this.ipfsSrc);
  },
};

</script>
