<template>
  <div id="app">
    <main>

      <div class="underlinedHeader">
        <h3 style="margin-bottom: 0; padding-left: 0.25em; position: relative; top: 0.5em;">{{ currentMonth }}</h3>
        <h1>Checkmate Speed Dating</h1>
        <hr>
      </div>

      <!--  -->

      <section style="display: flex;">
        <div style="flex-basis: calc(100%/3);">
          <p>Female</p>
          <ul>
            <li v-for="dater in participantsABC_Female" style="margin-bottom: 1em;">
              <b>{{ dater.name }}</b>, <small>{{ dater.orientation | f_reorient }}</small>
              <br><small style="position: relative; top: -0.5em;">({{ dater.emailCurrentMatches.length }} matches)</small>

              <ul>
                <li v-for="d in dater.emailCurrentMatches">{{d.name}}, <small>{{d.sex}}</small></li>
                <li v-for="d in dater.emailInvalidMatches" style=""><del>{{d.name}}, <small>{{d.sex}}</small></del></li>
              </ul>
            </li>
            <!--  -->

            <li v-if="!participantsABC_Female.length">No participants available.</li>
          </ul>
        </div>

        <div style="flex-basis: calc(100%/3);">
          <p>Male</p>
          <ul>
            <li v-for="dater in participantsABC_Male" style="margin-bottom: 1em;">
              <b>{{ dater.name }}</b>, <small>{{ dater.orientation | f_reorient }}</small>
              <br><small style="position: relative; top: -0.5em;">({{ dater.emailCurrentMatches.length }} matches)</small>

              <ul>
                <li v-for="d in dater.emailCurrentMatches">
                  {{d.name}}, <small>{{d.sex}}</small>
                </li>
                <li v-for="d in dater.emailInvalidMatches">
                  <del>{{d.name}}, <small>{{d.sex}}</small></del>
                </li>
              </ul>
            </li>
            <!--  -->

            <li v-if="!participantsABC_Male.length">No participants available.</li>
          </ul>
        </div>

        <div style="flex-basis: calc(100%/3);">
          <p>Non-Binary</p>
          <ul>
            <li v-for="dater in participantsABC_NonBinary" style="margin-bottom: 1em;">
              <b>{{ dater.name }}</b>, <small>{{ dater.orientation | f_reorient }}</small>
              <br><small style="position: relative; top: -0.5em;">({{ dater.emailCurrentMatches.length }} matches)</small>

              <ul>
                <li v-for="d in dater.emailCurrentMatches">{{d.name}}, <small>{{d.sex}}</small></li>
                <li v-for="d in dater.emailInvalidMatches" style=""><del>{{d.name}}, <small>{{d.sex}}</small></del></li>
              </ul>
            </li>
            <!--  -->

            <li v-if="!participantsABC_NonBinary.length">No participants available.</li>
          </ul>
        </div>
      </section>
      
      <!-- <div class="dater_cards">
        <div v-for="dater in participants" :key="dater.email">
          <h2>
            {{ dater.name }}
            <br><small>{{ dater.email }}</small>
          </h2>

          <p class="dater_info">
            <b>{{ dater.orientation | f_reorient }}</b> <b>{{ dater.sex }}</b>, {{ dater.pronouns | f_underscoreSlash }}
          </p>
          <p v-if="dater.comments">{{ dater.comments }}</p>


          <div style="margin-top: 0; display: flex;">
            <div style="border-right: 2px solid #fff; padding-top: 1em; flex-basis: 50%;">
              <p style="text-align: center;"><b>Matches</b> ({{ dater.emailCurrentMatches.length }})</p>
              <ul style="margin: 0; padding: 0.5em 1em 0.5em 1em">
                <li v-if="dater.emailCurrentMatches"
                  v-for="matches in dater.emailCurrentMatches"
                  :key="matches.index"
                >
                  {{ matches }}</li>
              </ul>
              <ul v-if="!dater.emailCurrentMatches.length" style="margin: 0;">
                <li>No matches this time.</li>
              </ul>
            </div>

            <div style="padding-top: 1em; flex-basis: 50%;">
              <p style="text-align: center;"><b>Invalid Matches</b> ({{ dater.emailInvalidMatches.length }})</p>
              <ul style="margin: 0; padding: 0.5em 0;">
                <li v-if="dater.emailInvalidMatches"
                  v-for="matches in dater.emailInvalidMatches"
                  :key="matches.index"
                  style="margin-left: 2em;"
                >
                  {{ matches }}</li>
              </ul>
              <ul v-if="!dater.emailInvalidMatches.length" style="margin: 0;">
                <li>Not applicable.</li>
              </ul>
            </div>
          </div>
        </div>
      </div> -->

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
      participantsABC_Current: [],
      participantsABC_Female: [],
      participantsABC_Male: [],
      participantsABC_NonBinary: [],
      queryResult: []
    }),
    apollo: {
      $loadingKey: 'loading',
      participants: {
        query: participants,
        result({data, loading, networkStatus}) {
          if(!loading) {
            const vm  = this;
            
            const participantsABC = vm.m_arrayAlphabetized(data.participants, 'name');

            vm.participantsABC_Current = participantsABC.filter(function(_dater) {
              if(_dater.attendance.includes(vm.currentEvent)) {
                return _dater;
              }
            });

            // console.log('vm.participantsABC_Current:', vm.participantsABC_Current);

            vm.participantsABC_Current.forEach(function(_dater) {

              _dater.emailCurrentMatches = [];
              _dater.emailInvalidMatches = [];

              vm.participantsABC_Current.filter(function(_partner) {
                // --------------------------------------------------------

                  switch(_dater.orientation.toLowerCase())
                  {
                    // -----------------------------------

                    case 'same':
                      if((_partner.orientation.toLowerCase() == 'same' && _partner.sex == _dater.sex) ||
                      (_partner.orientation.toLowerCase() == 'all' && _partner.sex == _dater.sex))
                      {
                        if(!_dater.email.includes(_partner.email) &&
                          !_dater.emailFriends.includes(_partner.email) &&
                          !_partner.emailFriends.includes(_dater.email) &&
                          !_dater.emailEncounters.includes(_partner.email) &&
                          !_partner.emailEncounters.includes(_dater.email))
                        {
                          return _dater.emailCurrentMatches.push(_partner);
                        }
                        else if(!_dater.email.includes(_partner.email))
                        {
                          return _dater.emailInvalidMatches.push(_partner);
                        }
                      }

                      break;
                    
                    case 'different':
                      if((_partner.orientation.toLowerCase() == 'different' && _partner.sex !== _dater.sex) ||
                        (_partner.orientation.toLowerCase() == 'all' && _partner.sex !== _dater.sex))
                      {
                        if(!_dater.email.includes(_partner.email) &&
                          !_dater.emailFriends.includes(_partner.email) &&
                          !_partner.emailFriends.includes(_dater.email) &&
                          !_dater.emailEncounters.includes(_partner.email) &&
                          !_partner.emailEncounters.includes(_dater.email))
                        {
                          return _dater.emailCurrentMatches.push(_partner);
                        }
                        else if(!_dater.email.includes(_partner.email))
                        {
                          return _dater.emailInvalidMatches.push(_partner);
                        }
                      }

                      break;
                    
                    case 'all':
                      if((_partner.orientation.toLowerCase() == 'all') ||
                      (_partner.orientation.toLowerCase() == 'same' && _partner.sex == _dater.sex) ||
                      (_partner.orientation.toLowerCase() == 'different' && _partner.sex !== _dater.sex))
                      {
                        if(!_dater.email.includes(_partner.email) &&
                          !_dater.emailFriends.includes(_partner.email) &&
                          !_partner.emailFriends.includes(_dater.email) &&
                          !_dater.emailEncounters.includes(_partner.email) &&
                          !_partner.emailEncounters.includes(_dater.email))
                        {
                          return _dater.emailCurrentMatches.push(_partner);
                        }
                        else if(!_dater.email.includes(_partner.email))
                        {
                          return _dater.emailInvalidMatches.push(_partner);
                        }
                      }

                    // -----------------------------------
                  }

                // --------------------------------------------------------
              });
            });

            vm.participantsABC_Female = vm.participantsABC_Current.filter(function(_dater) {
              if(_dater.sex == 'Female') {
                return vm.m_participantSexMatches(_dater);
              }
            });

            vm.participantsABC_Male = vm.participantsABC_Current.filter(function(_dater) {
              if(_dater.sex == 'Male') {
                return vm.m_participantSexMatches(_dater);
              }
            });

            vm.participantsABC_NonBinary = vm.participantsABC_Current.filter(function(_dater) {
              if(_dater.sex == 'NonBinary') {
                return vm.m_participantSexMatches(_dater);
              }
            });

          } // if() {}
        } // result() {}
      } // participants: {}
    },
    computed: {
      currentDate: function() {
        return new Date();
      },
      currentMonth: function() {
        return new Intl.DateTimeFormat('en-US', {month: 'long'}).format(this.currentDate);
      },
      currentYear: function() {
        return this.currentDate.getFullYear();
      },
      currentEvent: function() {
        console.log(this.currentMonth + '_' + this.currentYear);
        return this.currentMonth + '_' + this.currentYear;
      }
    },
    methods: {
      m_arrayAlphabetized: function(_data, _parameter) {
        // console.log('_data', _data);
        // console.log('_parameter', _parameter);

        return _data.sort(function(a, b) {

          // console.log('a:', a.name);
          // console.log('a.' + _parameter + ':', a[_parameter]);
          // console.log('b:', b);
          // console.log('b.' + _parameter + ':', b[_parameter]);

          const first = a[_parameter].toUpperCase();
          const second = b[_parameter].toUpperCase();

          if(first > second) {
            return 1;
          }

          if(first < second) {
            return -1;
          }

          return 0;
        });
      },
      m_participantSexMatches: function(_data) {
        // console.log(_data.name + ':');

        _data.emailCurrentMatches.forEach(function(_match) {
          // console.log(_match);
        })

        return _data;
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
    margin-top: 0;
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
    /* min-width: 333px; */
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