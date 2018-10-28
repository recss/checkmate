<template>
  <div id="app">
    <main>
      <div class="underlinedHeader">
        <h1>Checkmate Speed Dating</h1>
        <hr>
      </div>

      <!--  -->
      <div class="dater_cards">
        <!-- <div>aaa</div>
        <div>aaa</div>
        <div>aaa</div> -->

        <!-- <div>aaa</div>
        <div>aaa</div>
        <div>aaa</div> -->

        <div v-for="dater in participants" :key="dater.email">
          <h2>
            {{ dater.name }}
            <br><small>{{ dater.email }}</small>
          </h2>

          <p class="dater_info">
            <b>{{ dater.orientation | f_reorient }}</b> <b>{{ dater.sex }}</b>, {{ dater.pronouns | f_underscoreSlash }}
          </p>
          <p v-if="dater.comments">{{ dater.comments }}</p>

          <p style="margin-top: 1em; text-align: center;"><b>Matches</b> ({{ dater.emailCurrentMatches.length }})</p>
            <ul
              v-if="dater.emailCurrentMatches"
              v-for="matches in dater.emailCurrentMatches"
              :key="matches.index"
              style="margin: 0;"
            >
              <li>{{ matches }}</li>
              <!-- <li v-on:change="m_emailToName(matches)">{{ matches }}</li> -->
            </ul>
            <ul v-if="!dater.emailCurrentMatches.length" style="margin: 0;">
              <li>No matches this time.</li>
            </ul>
        </div>

        <!-- <div>aaa</div>
        <div>aaa</div>
        <div>aaa</div> -->

        <!-- <div>aaa</div>
        <div>aaa</div>
        <div>aaa</div> -->
      </div>

      <!-- <ul v-for="dater in participants" :key="dater.email">
        <li><b>{{ dater.name }}</b> ({{ dater.email }})</li>
        <ul>
          <li>Information</li>
          <ul>
            <li>{{ dater.sex }} ({{ dater.pronouns | f_underscoreSlash }})</li>
            <li>{{ dater.orientation | f_reorient }}</li>
            <li v-if="dater.comments">{{ dater.comments }}</li>
          </ul>

          <li>Friends</li>
          <ul v-for="friend in dater.emailFriends" :key="friend.index">
            <li>{{ friend }}</li>
          </ul>

          <li>Encounters</li>
          <ul v-for="encountered in dater.emailEncounters" :key="encountered.index">
            <li>{{ encountered }}</li>
          </ul>

          <li>Matches ({{ dater.emailCurrentMatches.length }})</li>
          <ul
            v-if="dater.emailCurrentMatches"
            v-for="matches in dater.emailCurrentMatches"
            :key="matches.index"
          >
            <li>{{ matches }}</li>
          </ul>
          <ul v-if="!dater.emailCurrentMatches.length">
            <li>No matches this time.</li>
          </ul>
        </ul>
      </ul> -->
    </main>
  </div>
</template>

<script>
  import gql from 'graphql-tag'

  const participants = gql`
    query participants {
      participants {
        id
        email
        name
        attendance
        emailFriends
        emailEncounters
        emailCurrentMatches
        sex
        orientation
        pronouns
        comments
      }
    }
  `

  export default {
    name: 'app',
    data: () => ({
      participants: [],
      queryResult: []
    }),
    apollo: {
      $loadingKey: 'loading',
      participants: {
        query: participants,
        result({data, loading, networkStatus}) {
          if(!loading) {
            console.log(data.participants);
            this.queryResult = data.participants;

            data.participants.forEach(function(_dater) {
              // console.log(_dater);

              _dater.emailCurrentMatches = [];

              return data.participants.filter(function(_partner) {
                // if not (yourself, friend, or encountered) and (sex and orientation) align, then connect

                // sex: female, male, nonbinary?
                // orientation: same, different, all?

                if(!_dater.email.includes(_partner.email) &&
                  !_dater.emailFriends.includes(_partner.email) &&
                  !_partner.emailFriends.includes(_dater.email) &&
                  !_dater.emailEncounters.includes(_partner.email))
                {
                  switch(_dater.orientation.toLowerCase())
                  {
                    case 'same':
                      if((_partner.orientation.toLowerCase() == 'same' && _partner.sex == _dater.sex) ||
                        (_partner.orientation.toLowerCase() == 'all' && _partner.sex == _dater.sex))
                      {
                        return _dater.emailCurrentMatches.push(_partner.email);
                      }
                      break;
                    
                    case 'different':
                      if((_partner.orientation.toLowerCase() == 'different' && _partner.sex !== _dater.sex) ||
                        (_partner.orientation.toLowerCase() == 'all' && _partner.sex !== _dater.sex))
                      {
                        return _dater.emailCurrentMatches.push(_partner.email);
                      }
                      break;
                    
                    case 'all':
                      if((_partner.orientation.toLowerCase() == 'all') ||
                        (_partner.orientation.toLowerCase() == 'same' && _partner.sex == _dater.sex) ||
                        (_partner.orientation.toLowerCase() == 'different' && _partner.sex !== _dater.sex))
                      {
                        return _dater.emailCurrentMatches.push(_partner.email);
                      }
                  }
                  // end of switch statement
                }
                // end of if statement
              });
            });
          }
        }
      }
    },
    methods: {
      m_emailToName: function(value) {
        console.log('just a minute too late', this.queryResult);
        return value;
      }
    },
    filters: {
      f_underscoreSlash: function(value) {
        if(!value) {
          return '';
        }

        const regex = /_/gi;

        return value.replace(regex, '/');
      },
      f_reorient: function(value) {
        if(!value) {
          return '';
        }

        switch(value) {
          case 'Same':
            value = 'Gay';
            break;
          
          case 'Different':
            value = 'Straight';
            break;
          
          case 'All':
            value = 'Bi/Pansexual';
            break;
        }

        return value;
      }
    }
  }
</script>

<style>
  *,
  *::before,
  *::after {
    box-sizing: border-box;
  }
  html {
    height: 100%;
  }
  body {
    min-height: 100vh;
    margin: 0;
    border: 1em solid rgba(254, 186, 239, 1);
  }
  #app {
    font-family: 'Source Sans Pro', sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    margin: 0;
    font-size: 16px;
    line-height: 1.5;
  }
  main {
    max-width: 1500px;
    margin: 2em auto;
    padding: 0 2em;
  }
  h1 {
    display: inline-block;
  }
  a {
    color: deepskyblue;
    text-decoration: none;
  }
  article {
    margin: 0 auto;
    max-width: 650px;
  }
  .underlinedHeader {
    width: max-content;
  }
  .underlinedHeader h1 {
    padding: 0 0.15em;
  }
  .underlinedHeader hr {
    border: 4px solid rgba(85, 253, 171, 0.7);
    position: relative;
      top: -3em;
    z-index: -10;
  }
  .dater_cards {
    display: flex;
      flex-flow: row wrap;
      justify-content: space-around;
      align-items: stretch;
  }
  .dater_cards > div {
    min-width: 333px;
    margin-bottom: 1em;
    border-radius: 0 1em;
    padding: 1em;
    background-color: rgba(254, 186, 239, 0.5);
    display: flex;
      flex-flow: column wrap;
      flex-basis: calc(100%/3.25);
  }
  .dater_cards > div h2 {
    margin: 0;
    padding: 0.45em;
    background-color: rgba(255,255,255,0.25);
    line-height: 1em;
    text-align: center;
  }
  .dater_cards > div small {
    font-size: 0.75em;
  }
  .dater_cards > div p {
    margin: 0;
  }
  .dater_info {
    padding-right: 0.25em;
    font-size: 0.9em;
    background-color: #fff;
    text-align: right;
  }
</style>