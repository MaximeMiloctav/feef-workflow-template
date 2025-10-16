<script setup lang="ts">
import type { SelectItem } from '@nuxt/ui';

interface Props {
  role?: 'oe' | 'feef' | 'auditeur'
}

const props = withDefaults(defineProps<Props>(), {
  role: 'feef'
})

const dossiersFinalisesParAn = [
  { annee: 2022, nombre: 18 },
  { annee: 2023, nombre: 27 },
  { annee: 2024, nombre: 31 },
  { annee: 2025, nombre: 8 }
]

const selectedAnnee = ref(dossiersFinalisesParAn.length > 0 ? dossiersFinalisesParAn[dossiersFinalisesParAn.length - 1].annee : 0)

// Données fictives pour la barre de progression détaillée
const dossierStats = {
  // En attente
  depotDossier: 5,
  validationFeef: 4, 
  signatureDevis: 5,
  // En cours
  planification: 8,
  audit: 9,
  finalisation: 5
}

const totalDossiers = Object.values(dossierStats).reduce((sum, val) => sum + val, 0)

// Calcul des pourcentages pour chaque étape
const depotPct = Math.round((dossierStats.depotDossier / totalDossiers) * 100)
const validationPct = Math.round((dossierStats.validationFeef / totalDossiers) * 100)
const devisPct = Math.round((dossierStats.signatureDevis / totalDossiers) * 100)
const planificationPct = Math.round((dossierStats.planification / totalDossiers) * 100)
const auditPct = Math.round((dossierStats.audit / totalDossiers) * 100)
const finalisationPct = 100 - depotPct - validationPct - devisPct - planificationPct - auditPct

// Totaux par catégorie principale
const totalAttente = dossierStats.depotDossier + dossierStats.validationFeef + dossierStats.signatureDevis
const totalEnCours = dossierStats.planification + dossierStats.audit + dossierStats.finalisation

const oeOptions = ref<SelectItem[]>([
  { label: 'Tous', value: 'all' },
  { label: 'SGS', value: 'sgs' },
  { label: 'Ecocert', value: 'ecocert' }
])
const selectedOE = ref(oeOptions.value[0].value)
// Données factices par catégorie et card
const dashboardCategories = [
  {
    label: 'Demande de dossiers',
    cards: [
      { shortText: 'Dépôt en cours', value: 7, alertesRouges: 0, alertesOranges: 0, auditInitial: 5, auditRenouvellement: 2, auditSuivi: 0, color: 'border-blue-500', bgColor: 'bg-blue-50' },
      { shortText: 'En attente de signature du contrat', value: 4, alertesRouges: 0, alertesOranges: 1, auditInitial: 2, auditRenouvellement: 1, auditSuivi: 1, color: 'border-orange-500', bgColor: 'bg-orange-50' },
      { shortText: 'En attente validation FEEF', value: 11, alertesRouges: 1, alertesOranges: 0, auditInitial: 8, auditRenouvellement: 3, auditSuivi: 0, color: 'border-green-500', bgColor: 'bg-green-50' },
    ]
  },
  // {
  //   label: 'Engagement',
  //   cards: [
  //     { shortText: 'En attente de devis', value: 5, alertesRouges: 0, alertesOranges: 2, auditInitial: 3, auditRenouvellement: 2, auditSuivi: 0, color: 'border-orange-500', bgColor: 'bg-orange-50' },
  //     { shortText: 'En attente de signature', value: 3, alertesRouges: 0, alertesOranges: 1, auditInitial: 1, auditRenouvellement: 1, auditSuivi: 1, color: 'border-orange-500', bgColor: 'bg-orange-50' },
  //     { shortText: 'En planification audit', value: 7, alertesRouges: 0, alertesOranges: 0, auditInitial: 4, auditRenouvellement: 2, auditSuivi: 1, color: 'border-green-500', bgColor: 'bg-green-50' }
  //   ]
  // },
  {
    label: 'Audit',
    cards: [
      { shortText: 'En attente d\'acceptation par l\'OE', value: 4, alertesRouges: 2, alertesOranges: 0, auditInitial: 4, auditRenouvellement: 0, auditSuivi: 0, color: 'border-green-500', bgColor: 'bg-green-50' },
      { shortText: 'En cours de planification', value: 7, alertesRouges: 0, alertesOranges: 0, auditInitial: 4, auditRenouvellement: 2, auditSuivi: 1, color: 'border-green-500', bgColor: 'bg-green-50' },
      { shortText: 'Audit planifié', value: 8, alertesRouges: 0, alertesOranges: 0, auditInitial: 3, auditRenouvellement: 2, auditSuivi: 3, color: 'border-blue-500', bgColor: 'bg-blue-50' },
      { shortText: 'Rapport attendu audit', value: 4, alertesRouges: 1, alertesOranges: 1, auditInitial: 1, auditRenouvellement: 1, auditSuivi: 2, color: 'border-blue-500', bgColor: 'bg-blue-50' }
    ]
  },
  {
    label: 'Decision',
    cards: [
      { shortText: 'Rapport envoyé', value: 2, alertesRouges: 0, alertesOranges: 1, auditInitial: 1, auditRenouvellement: 0, auditSuivi: 1, color: 'border-purple-500', bgColor: 'bg-purple-50' },
      { shortText: "Plan d'action en attente", value: 6, alertesRouges: 2, alertesOranges: 0, auditInitial: 2, auditRenouvellement: 2, auditSuivi: 2, color: 'border-orange-500', bgColor: 'bg-orange-50' },
      { shortText: 'En attente attestation', value: 1, alertesRouges: 0, alertesOranges: 0, auditInitial: 0, auditRenouvellement: 1, auditSuivi: 0, color: 'border-green-500', bgColor: 'bg-green-50' }
    ]
  }
];

const categoryTotals = dashboardCategories.map(cat => cat.cards.reduce((sum, card) => sum + card.value, 0));

</script>

<template>
    <div v-if="role === 'auditeur'" class="p-4 space-y-4">
        <!-- Vue spécifique auditeur -->
        <AuditsTable />
    </div>
    <div v-else class="p-4 space-y-4">
        <!-- Titre dashboard + select OE -->
        <div class="flex flex-row items-center justify-center w-full mb-1">
          <span class="font-bold text-3xl">Tableau de bord</span>
        </div>
        <div v-if="role === 'feef'" class="flex flex-row items-center gap-2 min-w-[220px] mb-1">
          <label class="font-semibold text-gray-700 whitespace-nowrap">Organisme évaluateur</label>
          <USelect v-model="selectedOE" :items="oeOptions" value-key="value" class="w-32" />
        </div>
        <USeparator class="mb-3" />
        <!-- Bloc infos entreprises, import et export -->
        <div class="flex flex-row gap-4 px-4 mb-4">
          <div class="bg-white rounded-xl shadow p-4 flex-1 flex flex-col items-center justify-center">
            <span class="text-gray-500 text-sm mb-1">Nombre d'entités</span>
            <span class="text-2xl font-bold text-primary-600">128</span>
          </div>
          <div class="bg-white rounded-xl shadow p-4 flex-1 flex flex-col items-center justify-center">
            <span class="text-gray-500 text-sm mb-1 text-center">Écart moyen audit planifié/réel</span>
            <span class="text-base font-semibold text-gray-800">+3,5 mois</span>
          </div>
          <div class="bg-white rounded-xl shadow p-4 flex-1 flex flex-col items-center justify-center">
            <span class="text-gray-500 text-sm mb-1 text-center">Durée moyenne du processus</span>
            <span class="text-base font-semibold text-gray-800">6 mois</span>
          </div>
          <div class="bg-white rounded-xl shadow p-4 flex-1 flex flex-col items-center justify-center">
            <span class="text-gray-500 text-sm mb-1 text-center">Durée moyenne d'acceptation par l'OE</span>
            <span class="text-base font-semibold text-gray-800">7,2 jours</span>
          </div>
          <template v-if="props.role === 'oe' || (selectedOE && selectedOE !== 'all')">
            <div class="bg-white rounded-xl shadow p-4 flex-1 flex flex-col items-center justify-center">
              <span class="text-gray-500 text-sm mb-1">Dernier import</span>
              <span class="text-base font-semibold text-gray-800">21/09/2025 à 14:32</span>
            </div>
            <div class="bg-white rounded-xl shadow p-4 flex-1 flex flex-col items-center justify-center">
              <span class="text-gray-500 text-sm mb-1">Dernier export</span>
              <span class="text-base font-semibold text-gray-800">20/09/2025 à 18:05</span>
            </div>
          </template>
        </div>
        <!-- Dossiers en cours (sans titre) -->
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-3 px-4 justify-start">
          <div v-for="(cat, i) in dashboardCategories" :key="cat.label" class="flex flex-col">
            <h2 class="text-lg font-bold text-gray-800 mb-2 text-center">
              <span class="text-primary-600">{{ categoryTotals[i] }}</span>
              &nbsp;{{ cat.label }}
            </h2>
            <div class="flex flex-col gap-3">
              <DashboardCard
                v-for="(card, j) in cat.cards"
                :key="j"
                v-bind="card"
                :lastValue="j % 3 === 0 ? undefined : card.value - (j % 2 === 0 ? 2 : -1)"
                :lastDate="j % 3 === 0 ? undefined : '20/09/2025 à 10:15'"
                :trend="j % 3 === 0 ? undefined : (card.value > (card.value - (j % 2 === 0 ? 2 : -1)) ? 'up' : 'down')"
              />
            </div>
          </div>
        </div>
        <!-- Barre de progression détaillée des dossiers -->
        <div class="w-full px-4 mt-6">
          <h2 class="text-lg font-bold mb-4 text-center">État des dossiers ({{ totalDossiers }} dossiers)</h2>
          
          <!-- Totaux par catégorie -->
          <div class="flex justify-center gap-8 mb-4">
            <div class="text-center">
              <div class="text-2xl font-bold text-orange-600">{{ totalAttente }}</div>
              <div class="text-sm text-gray-600">En attente</div>
            </div>
            <div class="text-center">
              <div class="text-2xl font-bold text-blue-600">{{ totalEnCours }}</div>
              <div class="text-sm text-gray-600">En cours</div>
            </div>
          </div>
          
          <!-- Barre de progression détaillée -->
          <div class="relative w-full h-8 bg-gray-100 rounded-full shadow-lg overflow-hidden flex text-xs font-medium">
            <!-- En attente - Dépôt dossier -->
            <div class="h-full flex items-center justify-center text-orange-900 transition-all duration-500 border-r border-white"
              :style="`width: ${depotPct}%; background: #fed7aa`">
              <span v-if="depotPct > 8" class="w-full text-center px-1">
                Dépôt {{ dossierStats.depotDossier }}
              </span>
            </div>
            
            <!-- En attente - Validation FEEF -->
            <div class="h-full flex items-center justify-center text-orange-900 transition-all duration-500 border-r border-white"
              :style="`width: ${validationPct}%; background: #fdba74`">
              <span v-if="validationPct > 8" class="w-full text-center px-1">
                Validation {{ dossierStats.validationFeef }}
              </span>
            </div>
            
            <!-- En attente - Signature devis -->
            <div class="h-full flex items-center justify-center text-orange-900 transition-all duration-500 border-r-2 border-gray-300"
              :style="`width: ${devisPct}%; background: #fb923c`">
              <span v-if="devisPct > 8" class="w-full text-center px-1">
                Devis {{ dossierStats.signatureDevis }}
              </span>
            </div>
            
            <!-- En cours - Planification -->
            <div class="h-full flex items-center justify-center text-blue-900 transition-all duration-500 border-r border-white"
              :style="`width: ${planificationPct}%; background: #bfdbfe`">
              <span v-if="planificationPct > 8" class="w-full text-center px-1">
                Planif. {{ dossierStats.planification }}
              </span>
            </div>
            
            <!-- En cours - Audit -->
            <div class="h-full flex items-center justify-center text-blue-900 transition-all duration-500 border-r border-white"
              :style="`width: ${auditPct}%; background: #93c5fd`">
              <span v-if="auditPct > 8" class="w-full text-center px-1">
                Audit {{ dossierStats.audit }}
              </span>
            </div>
            
            <!-- En cours - Finalisation -->
            <div class="h-full flex items-center justify-center text-blue-900 transition-all duration-500"
              :style="`width: ${finalisationPct}%; background: #60a5fa`">
              <span v-if="finalisationPct > 8" class="w-full text-center px-1">
                Final. {{ dossierStats.finalisation }}
              </span>
            </div>
          </div>
        </div>
        <USeparator class="my-4" />
        <!-- Bloc Statistiques -->
        <div>
          <div class="flex flex-row items-center gap-4 w-full">
            <div class="flex flex-col items-center w-1/2">
              <h2 class="text-lg font-bold mb-1 text-center">Audits prévus par mois</h2>
              <LineChartAudits />
            </div>
            <div class="flex flex-col items-center w-1/2">
              <h2 class="text-lg font-bold mb-1 text-center">Labellisés par année</h2>
              <BarChartLabellises />
            </div>
          </div>
        </div>
      </div>
</template>