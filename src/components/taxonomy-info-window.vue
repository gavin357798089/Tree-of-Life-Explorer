<template lang="pug">
.card
  .card-content
    .content
      .media
        .media-content
          h1.title.has-text-dark {{ scientificName }}
          h4.subtitle(v-if="commonName")
            span.has-text-grey='Commonly known as a '
            span.has-text-success {{ commonName }}

          .tag-list(v-if="!isMRCA")
            .tags.has-addons
              span.tag.is-dark
                | Rank
              a(:href="rankLink", target="_blank").tag.is-primary
                | {{ txnInfo.rank }}
            .tags.has-addons(v-if="txnInfo.numDescendants")
              span.tag.is-dark
                | Decendants
              span.tag.is-info
                | {{ txnInfo.numDescendants }}

          b-message(v-if="isMRCA", type="is-info")
            | This is an MRCA node
        .media-right
          .controls
            b-field(position="is-centered")
              .control
                .button.is-small.is-rounded(@click="prevImage")
                  b-icon(icon="menu-left")
              .control
                .button.is-small.is-rounded(@click="nextImage")
                  b-icon(icon="menu-right")
          .image
            img(:src="txnInfo.pic[currentPic]")
    .links
      .is-size-5.heading Find more information on...
      .columns
        .column(v-for="link in links")
          a(:href="link.url", target="_blank")
            .card
              .card-image
                figure.image
                  img(:src="link.image")
              .card-content
                .content.has-text-centered {{ link.text }}
</template>

<script>
import _remove from 'lodash/remove'
import { isMRCA } from '@/lib/taxonomy'
import { getTxnSourceId } from '@/lib/otol'

export default {
  name: 'TaxonomyInfoWindow'
  , props: {
    leaf: Object
    , txnInfo: Object
  }
  , data: () => ({
    currentPic: 0
  })
  , computed: {
    commonName(){
      if ( this.isMRCA ){ return '' }
      if ( !this.txnInfo || !this.txnInfo.vernacularNameList ){
        if ( this.otherCommonNames ){
          return this.otherCommonNames[0]
        }
        return ''
      }
      return this.txnInfo.vernacularNameList
    }
    , scientificName(){
      if ( !this.txnInfo ){ return '' }
      return this.txnInfo.canonicalName ||
        this.txnInfo.name ||
        this.leaf.node_id || ''
    }
    , isMRCA(){
      return this.leaf && isMRCA(this.leaf)
    }
    , rankLink(){
      return `https://en.wikipedia.org/wiki/${this.txnInfo.rank}_(biology)`
    }
    , links(){
      let ncbiId = getTxnSourceId( 'ncbi', this.leaf )
      let gbifId = getTxnSourceId( 'gbif', this.leaf )
      return _remove([
        {
          text: `Wikipedia`
          , url: this.isMRCA ? false : `https://en.wikipedia.org/wiki/${this.scientificName}`
          , image: 'https://3c1703fe8d.site.internapcdn.net/newman/gfx/news/hires/2017/58af0228b8aa8.jpg'
        }
        , {
          text: `Open Tree of Life`
          , url: `https://tree.opentreeoflife.org/opentree/argus/opentree9.1@${this.leaf.node_id}`
          , image: 'https://avatars2.githubusercontent.com/u/1752618?s=400&v=4'
        }
        , {
          text: 'NCBI'
          , url: ncbiId ? `https://www.ncbi.nlm.nih.gov/Taxonomy/Browser/wwwtax.cgi?mode=Info&id=${ncbiId}` : false
          , image: 'https://pbs.twimg.com/profile_images/759051827/26243_367877492480_367794192480_4194340_6124010_n_400x400.jpg'
        }
        , {
          text: 'GBIF'
          , url: gbifId ? `https://www.gbif.org/species/${gbifId}` : false
          , image: 'https://pbs.twimg.com/profile_images/638655521218785280/XCFoWw5r.jpg'
        }
      ], obj => obj.url)
    }
  }
  , components: {
  }
  , methods: {
    nextImage(){
      let len = this.txnInfo.pic.length
      this.currentPic = (this.currentPic + 1) % len
    }
    , prevImage(){
      let len = this.txnInfo.pic.length
      this.currentPic = (this.currentPic - 1 + len) % len
    }
  }
}
</script>

<style lang="sass" scoped>
@import '@/styles/_variables.scss'
.media-right
  position: relative
  width: 256px
  .controls
    position: absolute
    top: 10px
    right: 10px
    z-index: 1
.tag-list
  display: flex
  .tags
    margin: 0 1em 0 0
.links
  .column
    max-width: 25%
  .card
    transition: all 0.15s ease-in-out
    &:hover,
    &:focus
      box-shadow: 0 2px 3px transparentize($blue, 0.6), 0 0 0 1px transparentize($blue, 0.6)
</style>