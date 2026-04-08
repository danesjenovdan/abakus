<template>
  <div id="builder">
    <div id="dz-container">
      <d-z-vec :num-highlighted="numHighlighted"></d-z-vec>
      <div id="info-container">
        <p
          v-if="numHighlighted < 46"
          id="prompt"
        >{{ prompt }}</p>
        <p v-else id="prompt">To je sicer dovolj za sestavo vlade, <strong>ampak ...</strong></p>
      </div>
    </div>
    <div id="party-buttons">
      <div class="party" v-for="party in parties" :key="party.acronym">
        <party-button
          class="bigone"
          :caption="party.acronym"
          :selected="selectedParties.indexOf(party.acronym) !== -1"
          @click.native="clickParty(party)"
        />
        <party-button
          v-for="person in party.people"
          :key="person.slug"
          :caption="person.name"
          :selected="selectedPeople.indexOf(person.slug) !== -1"
          @click.native="clickPerson(person)"
        ></party-button>
      </div>
    </div>
  </div>
</template>

<script>
import PartyButton from './PartyButton.vue';
import DZVec from './DZ.vue';

export default {
  name: 'Builder',

  components: {
    PartyButton,
    DZVec,
  },

  data() {
    return {
      numHighlighted: 0,
      people: [],
      parties: [],
      selectedParties: [],
      selectedPeople: [],
      chosenParties: [],
    };
  },

  created() {
    fetch('https://parladata-slovenija.lb.djnd.si/v3/cards/misc/members/?id=137&members:per_page=90')
      .then((response) => {
        if (!response.ok) throw new Error(`HTTP error: ${response.status}`);
        return response.json();
      })
      .then((dz) => {
        const partiesDict = dz.results.members.reduce(
          (acc, person) => {
            const acronym = person.group?.acronym ?? 'Brez poslanske skupine';
            (acc[acronym] = acc[acronym] || []).push(person);
            return acc;
          },
          {},
        );

        this.parties = Object.keys(partiesDict).map((key) => ({
          people: partiesDict[key],
          acronym: key,
          count: partiesDict[key].length,
        }));

        this.people = dz.results.members.map((person) => ({
          acronym: person.name,
          seats: 1,
          selected: false,
          loserImg: '',
          loserText: '',
        }));
      });
  },

  computed: {
    prompt() {
      switch (this.numHighlighted) {
        case 0:
          return 'Izberi stranke, ki bi po tvojem lahko sestavile koalicijo.';
        default:
          return this.numHighlighted > 45 ? 'REZULTAT' : 'To je premalo za sestavo koalicije.';
      }
    },
  },

  methods: {
    clickParty(party) {
      const index = this.selectedParties.indexOf(party.acronym);
      if (index === -1) {
        this.parties.filter((p) => p.acronym === party.acronym)[0].people.forEach((person) => {
          const index2 = this.selectedPeople.indexOf(person.slug);
          if (index2 === -1) {
            this.selectedPeople.push(person.slug);
            this.numHighlighted += 1;
          }
        });
        this.selectedParties.push(party.acronym);
      } else {
        this.parties.filter((p) => p.acronym === party.acronym)[0].people.forEach((person) => {
          const index2 = this.selectedPeople.indexOf(person.slug);
          if (index2 !== -1) {
            this.selectedPeople.splice(index2, 1);
            this.numHighlighted -= 1;
          }
        });
        this.selectedParties.splice(index, 1);
      }

      // if (this.numHighlighted > 45) {
      //   this.chooseParties();
      // }
    },
    clickPerson(person) {
      const index = this.selectedPeople.indexOf(person.slug);
      if (index === -1) {
        this.selectedPeople.push(person.slug);
        this.numHighlighted += 1;
      } else {
        this.selectedPeople.splice(index, 1);
        this.numHighlighted -= 1;
      }

      // if (this.numHighlighted > 45) {
      //   this.chooseParties();
      // }
    },
    chooseParties() {
      const firstindex = Math.floor(Math.random() * this.selectedParties.length);
      const secondindex = Math.floor(Math.random() * this.selectedParties.length);
      if (firstindex !== secondindex) {
        this.chosenParties = this.parties.filter((party) => (
          party.acronym === this.selectedParties[firstindex]
        ) || (party.acronym === this.selectedParties[secondindex])) // eslint-disable-line
      } else {
        this.chooseParties();
      }
    },

    clear() {
      this.numHighlighted = 0;
      this.selectedParties = [];
    },

    flipAndGo() {
      document.getElementsByTagName('body')[0].className += ' rotate';
      // document.getElementsByTagName('body')[0].style.transform = 'rotate(180deg)';
      setTimeout(() => {
        this.$router.push({ path: 'oglas/0' });
      }, 1350);

      setTimeout(() => {
        document.getElementsByTagName('body')[0].className = '';
      }, 2000);
    },
  },
};
</script>

<style lang="scss" scoped>
#builder {
  display: block;
  // padding-top: 40px;
  flex-wrap: wrap;
  text-align: left;
  overflow: hidden;

  #party-buttons {
    flex: 1;
    width: 45%;
    float: left;

    @media (max-width: 768px) {
      width: 95%;
    }
  }

  .regular-button {
    margin-bottom: 10px;
  }

  #dz-container {
    position: fixed;
    display: block;
    width: 50%;
    right: 10px;

    padding: 20px;

    text-align: center;

    svg {
      max-height: 347px;
    }

    @media (max-width: 768px) {
      // display: none;
      width: 100%;
      display: block;
      position: relative;
    }
  }

  #info-container {
    #prompt {
      width: 100%;
      text-align: center;
      margin: auto;
      margin-top: 30px;
    }

    .tiny-card-container {
      width: 100%;
      display: flex;

      @media (max-width: 768px) {
        flex-wrap: wrap;

        .tiny-card {
          margin: 0;
        }
      }
    }

    @media (max-width: 768px) {
      margin-left: 20px;
    }
  }

  #prompt {
    max-width: 400px;
    text-align: left;
    color: #070c2f;
    font-family: 'Barlow', sans-serif;
    font-size: 24px;
    font-weight: 300;
    line-height: 30px;

    margin-top: 0;

    strong {
      font-weight: 700;
    }
  }

  .button-container {
    width: 100%;
    text-align: left;
    position: relative;
    margin-top: 20px;
  }
}
</style>

<style lang="scss">
.rotate {
  transition: all 1.5s ease-in;
  transform: rotate(360deg);

  #footer {
    display: none;
  }
}

.bigone {
  width: 100% !important;
  margin-top: 30px !important;
}
</style>
