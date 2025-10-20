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

// Fonction pour calculer la priorité et les couleurs basées sur la date limite
const getPrioriteAction = (dateLimite: string) => {
  const aujourdhui = new Date()
  const dateL = new Date(dateLimite)
  const diffJours = Math.ceil((dateL.getTime() - aujourdhui.getTime()) / (1000 * 3600 * 24))

  if (diffJours < 0) {
    // Date dépassée - ROUGE
    return {
      priorite: 3,
      color: 'border-red-500',
      bgColor: 'bg-red-50'
    }
  } else if (diffJours <= 7) {
    // Moins d'une semaine - ORANGE
    return {
      priorite: 2,
      color: 'border-orange-500',
      bgColor: 'bg-orange-50'
    }
  } else {
    // Plus d'une semaine - VERT
    return {
      priorite: 1,
      color: 'border-green-500',
      bgColor: 'bg-green-50'
    }
  }
}

// Données brutes des actions par acteur (avec dates limites)
const actionsParActeurRaw = {
  feef: [
    { title: 'Valider le dépôt d\'un dossier', linkedTo: 'SARL Boulangerie Martin', linkedToType: 'entity' as const, dateLimite: '2025-10-18' }, // Date dépassée
    { title: 'Valider le dépôt d\'un dossier', linkedTo: 'SAS TechStart', linkedToType: 'entity' as const, dateLimite: '2025-10-24' }, // Dans 4 jours
    { title: 'Valider la labellisation d\'un Audit', linkedTo: 'Audit #2024-A-0125', linkedToType: 'audit' as const, dateLimite: '2025-10-19' }, // Date dépassée
    { title: 'Valider la labellisation d\'un Audit', linkedTo: 'Audit #2024-A-0134', linkedToType: 'audit' as const, dateLimite: '2025-11-05' }, // Dans 16 jours
    { title: 'Valider le dépôt d\'un dossier', linkedTo: 'EURL Garage Dupont', linkedToType: 'entity' as const, dateLimite: '2025-10-26' } // Dans 6 jours
  ],
  oe: [
    { title: 'Accepter un audit', linkedTo: 'Audit #2025-A-0012', linkedToType: 'audit' as const, dateLimite: '2025-10-17' }, // Date dépassée
    { title: 'Accepter un audit', linkedTo: 'Audit #2025-A-0015', linkedToType: 'audit' as const, dateLimite: '2025-10-22' }, // Dans 2 jours
    { title: 'Mettre en ligne le plan et les dates d\'audit', linkedTo: 'Audit #2024-A-0198', linkedToType: 'audit' as const, dateLimite: '2025-11-10' }, // Dans 21 jours
    { title: 'Mettre en ligne le rapport d\'audit et les scores', linkedTo: 'Audit #2024-A-0156', linkedToType: 'audit' as const, dateLimite: '2025-10-25' }, // Dans 5 jours
    { title: 'Valider le plan d\'action correctif', linkedTo: 'Audit #2024-A-0142', linkedToType: 'audit' as const, dateLimite: '2025-11-02' }, // Dans 13 jours
    { title: 'Mettre en ligne l\'avis de labellisation', linkedTo: 'Audit #2024-A-0133', linkedToType: 'audit' as const, dateLimite: '2025-10-16' } // Date dépassée
  ],
  entites: [
    { title: 'Déposer son dossier', linkedTo: 'SARL Martin & Fils', linkedToType: 'entity' as const, dateLimite: '2025-10-23' }, // Dans 3 jours
    { title: 'Mettre une nouvelle version des documents demandés', linkedTo: 'SAS InnoTech', linkedToType: 'entity' as const, dateLimite: '2025-10-15' }, // Date dépassée
    { title: 'Signer un contrat avec la FEEF', linkedTo: 'EURL Services Plus', linkedToType: 'entity' as const, dateLimite: '2025-11-08' }, // Dans 19 jours
    { title: 'Mettre en ligne son plan d\'action correctif', linkedTo: 'SA TechGroup', linkedToType: 'entity' as const, dateLimite: '2025-10-27' }, // Dans 7 jours
    { title: 'Mettre à jour les informations de son dossier', linkedTo: 'SARL Boulangerie Martin', linkedToType: 'entity' as const, dateLimite: '2025-10-21' } // Dans 1 jour
  ]
}

// Traitement des actions avec calcul de priorité et tri
const actionsParActeur = computed(() => {
  const processActions = (actions: typeof actionsParActeurRaw.feef) => {
    return actions
      .map(action => {
        const priorite = getPrioriteAction(action.dateLimite)
        return {
          ...action,
          ...priorite
        }
      })
      .sort((a, b) => b.priorite - a.priorite) // Tri par priorité décroissante (rouge > orange > vert)
  }

  return {
    feef: processActions(actionsParActeurRaw.feef),
    oe: processActions(actionsParActeurRaw.oe),
    entites: processActions(actionsParActeurRaw.entites)
  }
})

// Gestion des onglets
const tabs = [
  { value: 'etapes', label: 'Étapes des dossiers', slot: 'etapes' },
  { value: 'actions', label: 'Actions', slot: 'actions' }
]

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
        <!-- Système d'onglets pour Étapes des dossiers et Actions -->
        <div class="px-4">
          <UTabs :items="tabs" :default-value="'etapes'">
            <!-- Onglet Étapes des dossiers -->
            <template #etapes>
              <div class="pt-4">
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-3 justify-start">
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
              </div>
            </template>

            <!-- Onglet Actions -->
            <template #actions>
              <div class="pt-4">
                <div class="grid grid-cols-1 lg:grid-cols-3 gap-4">
                  <!-- Colonne Actions FEEF -->
                  <div class="flex flex-col">
                    <h2 class="text-lg font-bold text-gray-800 mb-3 text-center">
                      <span class="text-primary-600">{{ actionsParActeur.feef.length }}</span>
                      &nbsp;Actions FEEF
                    </h2>
                    <div class="flex flex-col gap-3">
                      <ActionCard
                        v-for="(action, idx) in actionsParActeur.feef"
                        :key="idx"
                        :title="action.title"
                        :linked-to="action.linkedTo"
                        :linked-to-type="action.linkedToType"
                        :date-limite="action.dateLimite"
                        :color="action.color"
                        :bg-color="action.bgColor"
                      />
                    </div>
                  </div>

                  <!-- Colonne Actions OE -->
                  <div class="flex flex-col">
                    <h2 class="text-lg font-bold text-gray-800 mb-3 text-center">
                      <span class="text-primary-600">{{ actionsParActeur.oe.length }}</span>
                      &nbsp;Actions OE
                    </h2>
                    <div class="flex flex-col gap-3">
                      <ActionCard
                        v-for="(action, idx) in actionsParActeur.oe"
                        :key="idx"
                        :title="action.title"
                        :linked-to="action.linkedTo"
                        :linked-to-type="action.linkedToType"
                        :date-limite="action.dateLimite"
                        :color="action.color"
                        :bg-color="action.bgColor"
                      />
                    </div>
                  </div>

                  <!-- Colonne Actions Entités -->
                  <div class="flex flex-col">
                    <h2 class="text-lg font-bold text-gray-800 mb-3 text-center">
                      <span class="text-primary-600">{{ actionsParActeur.entites.length }}</span>
                      &nbsp;Actions Entités
                    </h2>
                    <div class="flex flex-col gap-3">
                      <ActionCard
                        v-for="(action, idx) in actionsParActeur.entites"
                        :key="idx"
                        :title="action.title"
                        :linked-to="action.linkedTo"
                        :linked-to-type="action.linkedToType"
                        :date-limite="action.dateLimite"
                        :color="action.color"
                        :bg-color="action.bgColor"
                      />
                    </div>
                  </div>
                </div>
              </div>
            </template>
          </UTabs>
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