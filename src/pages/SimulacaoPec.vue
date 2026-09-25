<template>
  <v-container class="pa-4 pa-md-8 bg-background" fluid style="min-height: 100vh">
    <!-- Aviso: deixa claro que é uma demonstração, não o PEC oficial -->
    <v-alert
      class="mb-6"
      color="primary"
      density="compact"
      icon="mdi-information-outline"
      variant="tonal"
    >
      Simulação: esta tela imita a busca de cidadão do e-SUS PEC para mostrar
      onde o risco familiar do e-TET apareceria. Não é o sistema oficial.
    </v-alert>

    <!-- "Janela" que imita o PEC -->
    <div class="pec-window">
      <!-- Barra superior azul, como no PEC -->
      <div class="pec-topbar">
        <span class="pec-brand">e-SUS APS <strong>PEC</strong></span>
        <span class="pec-user">Profissional de saúde (simulação)</span>
      </div>

      <!-- Trilha de navegação -->
      <div class="pec-breadcrumb">
        <v-icon class="mr-1" color="primary" size="18">mdi-apps</v-icon>
        <span class="text-primary font-weight-bold">Módulos</span>
        <v-icon class="mx-2" color="grey" size="16">mdi-chevron-right</v-icon>
        <span>Cidadão</span>
      </div>

      <!-- Filtros de busca -->
      <div class="pec-section pec-section--white">
        <h1 class="pec-title">Cidadão</h1>

        <label class="pec-label" for="pec-busca">Nome/CNS/CPF do cidadão</label>
        <v-text-field
          id="pec-busca"
          v-model="search"
          class="mb-4"
          clearable
          density="comfortable"
          hide-details
          rounded="0"
          variant="outlined"
          @keyup.enter="performSearch"
        />

        <v-row dense>
          <v-col cols="12" md="2" sm="4">
            <label class="pec-label" for="pec-nascimento">Data de nascimento</label>
            <v-text-field
              id="pec-nascimento"
              v-model="birthDateFilter"
              v-maska="'##/##/####'"
              append-inner-icon="mdi-calendar-blank-outline"
              density="compact"
              hide-details
              placeholder="dd/mm/aaaa"
              rounded="0"
              variant="outlined"
            />
          </v-col>
          <v-col cols="12" md="5" sm="8">
            <label class="pec-label" for="pec-mae">Nome da mãe</label>
            <v-text-field
              id="pec-mae"
              v-model="motherNameFilter"
              density="compact"
              hide-details
              rounded="0"
              variant="outlined"
            />
          </v-col>
          <v-col cols="12" md="5">
            <label class="pec-label" for="pec-municipio">Município de nascimento</label>
            <v-select
              id="pec-municipio"
              v-model="municipalityFilter"
              clearable
              density="compact"
              hide-details
              :items="municipalities"
              no-data-text="Nenhum município nos dados sincronizados"
              rounded="0"
              variant="outlined"
            />
          </v-col>
        </v-row>

        <div class="d-flex justify-end flex-wrap ga-3 mt-6">
          <v-btn rounded="sm" variant="outlined" @click="clearFilters">
            Limpar filtros
          </v-btn>
          <v-btn
            color="primary"
            prepend-icon="mdi-magnify"
            rounded="sm"
            variant="flat"
            @click="performSearch"
          >
            Buscar cidadão
          </v-btn>
        </div>
      </div>

      <!-- Resultados -->
      <div class="pec-section">
        <div class="d-flex align-baseline flex-wrap ga-3 mb-4">
          <h2 class="pec-subtitle">Cadastros na base local</h2>
          <span v-if="hasSearched" class="text-body-2">
            {{ results.length }}
            {{ results.length === 1 ? 'cidadão encontrado' : 'cidadãos encontrados' }}
          </span>

          <!-- Estado da consulta ao e-TET, para a conversa ficar visível -->
          <span v-if="fetchStatus === 'loading'" class="pec-status">
            <v-progress-circular class="mr-2" indeterminate size="14" width="2" />
            Consultando a classificação de risco no e-TET...
          </span>
          <span v-else-if="fetchStatus === 'done'" class="pec-status pec-status--ok">
            <v-icon class="mr-1" size="16">mdi-check-circle-outline</v-icon>
            Classificações recebidas do e-TET às {{ fetchedAt }}
          </span>
          <span v-else-if="fetchStatus === 'error'" class="pec-status pec-status--off">
            <v-icon class="mr-1" size="16">mdi-alert-circle-outline</v-icon>
            O e-TET não respondeu. Mostrando a última cópia local.
          </span>
        </div>

        <v-alert
          v-if="fetchStatus === 'done' && usedFallback"
          class="mb-4"
          density="compact"
          icon="mdi-alert-outline"
          type="warning"
          variant="tonal"
        >
          Algumas famílias não responderam. Para elas, o valor mostrado é a última cópia local.
        </v-alert>

        <div v-if="!hasSearched" class="pec-empty">
          Digite o nome, CPF ou CNS e clique em "Buscar cidadão".
        </div>

        <div v-else-if="results.length === 0" class="pec-empty">
          Nenhum cidadão encontrado com esses filtros. Confira os dados ou
          sincronize na tela "Cidadãos".
        </div>

        <div v-for="citizen in results" :key="citizen.id" class="pec-card">
          <div class="flex-grow-1">
            <!-- O "pulo do gato": risco da família ao lado do nome -->
            <div class="d-flex align-center flex-wrap ga-2 mb-2">
              <span class="pec-name">{{ citizen.name }}</span>
              <v-tooltip location="top">
                <template #activator="{ props: tooltipProps }">
                  <v-chip
                    v-bind="tooltipProps"
                    class="font-weight-bold"
                    :color="getRiskColor(familyRisk(citizen).label)"
                    label
                    prepend-icon="mdi-home-heart"
                    size="small"
                    variant="flat"
                  >
                    {{ familyRisk(citizen).label }}
                  </v-chip>
                </template>
                <span>{{ familyRisk(citizen).tooltip }}</span>
              </v-tooltip>
            </div>

            <div class="pec-grid">
              <div><b>CPF</b> {{ formatCpf(citizen.cpf) }}</div>
              <div><b>Nome da mãe</b> {{ citizen.motherName || '-' }}</div>
              <div><b>CNS</b> {{ citizen.cns || '-' }}</div>
              <div><b>Telefone</b> {{ citizen.phone || '-' }}</div>
              <div>
                <b>Data de nascimento</b> {{ formatDate(citizen.birthDate) }}
                <span v-if="citizen.birthDate">({{ calculateAge(citizen.birthDate) }} anos)</span>
              </div>
              <div><b>Unidade responsável</b> {{ citizen.responsibleUnit || '-' }}</div>
              <div><b>Sexo</b> {{ citizen.sex || '-' }}</div>
              <div><b>Município de nascimento</b> {{ citizen.birthMunicipality || '-' }}</div>
              <div><b>Última atualização</b> {{ formatDate(citizen.lastUpdate || '') }}</div>
            </div>
          </div>

          <div class="d-flex align-start ga-1">
            <v-btn
              prepend-icon="mdi-magnify"
              rounded="sm"
              size="small"
              variant="outlined"
              @click="goToDetail(citizen.id)"
            >
              Visualizar
            </v-btn>
          </div>
        </div>

        <h2 class="pec-subtitle mt-8 mb-4">Cadastros na base nacional</h2>
        <div class="pec-national">
          <v-icon class="mr-2" size="18">mdi-magnify</v-icon>
          Pesquisar na base nacional (indisponível na simulação)
        </div>
      </div>
    </div>
  </v-container>
</template>

<script setup lang="ts">
  import type { ICitizen, IFamily } from '@/types/citizen'
  import { useObservable } from '@vueuse/rxjs'
  import { liveQuery } from 'dexie'
  import { computed, ref } from 'vue'
  import { useRouter } from 'vue-router'
  import api from '@/services/api'
  import { db } from '@/services/localDb'

  const router = useRouter()

  // Filtros (mesmos campos da tela Cidadão do PEC)
  const search = ref('')
  const birthDateFilter = ref('')
  const motherNameFilter = ref('')
  const municipalityFilter = ref<string | null>(null)

  // Como no PEC, o resultado só aparece depois de clicar em "Buscar cidadão"
  const hasSearched = ref(false)
  const results = ref<ICitizen[]>([])

  // Classificações recebidas do e-TET nesta consulta
  const etetRisks = ref(new Map<string, IFamily>())
  const fetchStatus = ref<'idle' | 'loading' | 'done' | 'error'>('idle')
  const fetchedAt = ref('')
  const usedFallback = ref(false)

  // Dados locais (já sincronizados pela tela "Cidadãos")
  const citizensRaw = useObservable<ICitizen[]>(
    liveQuery(() => db.citizens.toArray()) as any,
  )
  const familiesRaw = useObservable<IFamily[]>(
    liveQuery(() => db.families.toArray()) as any,
  )

  const familiesMap = computed(() => {
    const map = new Map<string, IFamily>()
    for (const f of (familiesRaw.value || [])) map.set(f.id, f)
    return map
  })

  const municipalities = computed(() => {
    const names = (citizensRaw.value || [])
      .map(c => c.birthMunicipality)
      .filter((m): m is string => !!m)
    return [...new Set(names)]
  })

  function performSearch () {
    let result = citizensRaw.value || []
    const term = (search.value || '').trim()

    if (term) {
      const numbers = term.replace(/\D/g, '')
      const lower = term.toLowerCase()
      result = result.filter(c => {
        const matchName = c.name.toLowerCase().includes(lower)
        const matchCpf = !!numbers && !!c.cpf && c.cpf.replace(/\D/g, '').includes(numbers)
        const matchCns = !!numbers && !!c.cns && c.cns.includes(numbers)
        return matchName || matchCpf || matchCns
      })
    }

    if (birthDateFilter.value && birthDateFilter.value.length === 10) {
      const [day, month, year] = birthDateFilter.value.split('/')
      result = result.filter(c => c.birthDate?.startsWith(`${year}-${month}-${day}`))
    }

    if (motherNameFilter.value) {
      const lower = motherNameFilter.value.toLowerCase()
      result = result.filter(c => c.motherName?.toLowerCase().includes(lower))
    }

    if (municipalityFilter.value) {
      result = result.filter(c => c.birthMunicipality === municipalityFilter.value)
    }

    results.value = result
    hasSearched.value = true

    fetchRisksFromEtet(result)
  }

  // Aqui acontece a "conversação": a tela pede ao e-TET a classificação
  // de cada família mostrada, no momento em que o resultado aparece.
  async function fetchRisksFromEtet (citizens: ICitizen[]) {
    const familyIds = [...new Set(citizens.map(c => c.familyId).filter((id): id is string => !!id))]

    if (familyIds.length === 0) {
      fetchStatus.value = 'idle'
      return
    }

    fetchStatus.value = 'loading'
    usedFallback.value = false

    const responses = await Promise.all(
      familyIds.map(async id => {
        try {
          const { data } = await api.get(`/families/${id}`)
          return { id, family: data as IFamily }
        } catch {
          return { id, family: null }
        }
      }),
    )

    const map = new Map<string, IFamily>()
    for (const { id, family } of responses) {
      if (family) {
        map.set(id, family)
      } else {
        usedFallback.value = true
      }
    }

    etetRisks.value = map
    fetchStatus.value = map.size > 0 ? 'done' : 'error'
    fetchedAt.value = new Date().toLocaleTimeString('pt-BR')
  }

  function clearFilters () {
    search.value = ''
    birthDateFilter.value = ''
    motherNameFilter.value = ''
    municipalityFilter.value = null
    results.value = []
    hasSearched.value = false
  }

  // Risco mostrado ao lado do nome: o da FAMÍLIA (vale para todos os moradores)
  function familyRisk (citizen: ICitizen) {
    const id = citizen.familyId

    // 1º: o que o e-TET respondeu agora
    const fromEtet = id ? etetRisks.value.get(id) : null
    if (fromEtet?.riskClass) {
      const score = fromEtet.riskScore === undefined ? '' : ` (escore ${fromEtet.riskScore})`
      return {
        label: fromEtet.riskClass,
        tooltip: `Classificação recebida do e-TET${score}`,
      }
    }

    // 2º: a cópia local, caso a API não responda
    const local = id ? familiesMap.value.get(id) : null
    if (local?.riskClass) {
      const score = local.riskScore === undefined ? '' : ` (escore ${local.riskScore})`
      return {
        label: local.riskClass,
        tooltip: `Cópia local, o e-TET não respondeu${score}`,
      }
    }

    if (citizen.riskClass) {
      return { label: citizen.riskClass, tooltip: 'Risco registrado no cadastro do cidadão' }
    }

    return { label: 'Sem classificação', tooltip: 'A família ainda não foi estratificada' }
  }

  function getRiskColor (risk?: string) {
    if (!risk) return 'grey'
    const r = risk.toUpperCase()
    if (r.includes('MÁXIMO') || r.includes('MAXIMO') || r.includes('R3')) return 'red-darken-4'
    if (r.includes('MÉDIO') || r.includes('MEDIO') || r.includes('R2')) return 'deep-orange-darken-2'
    if (r.includes('MENOR') || r.includes('R1')) return 'orange-darken-2'
    if (r.includes('BAIXO') || r.includes('R0') || r.includes('SEM RISCO')) return 'green-darken-2'
    return 'grey'
  }

  function goToDetail (id: string) {
    router.push(`/citizen/${id}`)
  }

  function formatCpf (cpf?: string) {
    if (!cpf) return '-'
    const d = cpf.replace(/\D/g, '')
    if (d.length !== 11) return cpf
    return `${d.slice(0, 3)}.${d.slice(3, 6)}.${d.slice(6, 9)}-${d.slice(9)}`
  }

  function formatDate (dateStr: string) {
    if (!dateStr) return '-'
    const date = new Date(dateStr.length === 10 ? `${dateStr}T00:00:00` : dateStr)
    return Number.isNaN(date.getTime()) ? '-' : date.toLocaleDateString('pt-BR')
  }

  function calculateAge (birthDate: string) {
    const birth = new Date(`${birthDate}T00:00:00`)
    const today = new Date()
    let age = today.getFullYear() - birth.getFullYear()
    const m = today.getMonth() - birth.getMonth()
    if (m < 0 || (m === 0 && today.getDate() < birth.getDate())) age--
    return age
  }
</script>

<style scoped>
.pec-window {
  border: 1px solid #d6d9de;
  background: #f0f1f4;
  max-width: 1100px;
  margin: 0 auto;
}

.pec-topbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 8px;
  background: #0069d0;
  color: #fff;
  padding: 10px 20px;
}

.pec-brand {
  font-size: 1.05rem;
}

.pec-user {
  font-size: 0.85rem;
  opacity: 0.9;
}

.pec-breadcrumb {
  display: flex;
  align-items: center;
  background: #fff;
  border-bottom: 1px solid #d6d9de;
  padding: 8px 20px;
  font-size: 0.875rem;
}

.pec-section {
  padding: 24px 20px;
}

.pec-section--white {
  background: #fff;
}

.pec-title {
  font-size: 1.5rem;
  font-weight: 700;
  margin-bottom: 20px;
}

.pec-subtitle {
  font-size: 1.25rem;
  font-weight: 700;
}

.pec-label {
  display: block;
  font-size: 0.8rem;
  font-weight: 700;
  margin-bottom: 4px;
}

.pec-card {
  display: flex;
  justify-content: space-between;
  gap: 16px;
  background: #fff;
  border: 1px solid #d6d9de;
  padding: 16px;
  margin-bottom: 8px;
}

.pec-name {
  font-weight: 700;
  font-size: 1rem;
}

.pec-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 2px 32px;
  font-size: 0.8rem;
}

.pec-grid b {
  margin-right: 4px;
}

.pec-status {
  display: inline-flex;
  align-items: center;
  font-size: 0.8rem;
  color: #555b66;
}

.pec-status--ok {
  color: #1b7a3e;
}

.pec-status--off {
  color: #b3541e;
}

.pec-empty {
  background: #fff;
  border: 1px dashed #c3c7cf;
  padding: 20px;
  font-size: 0.9rem;
  color: #555b66;
}

.pec-national {
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fff;
  border: 1px solid #d6d9de;
  padding: 14px;
  font-size: 0.875rem;
  color: #8a909b;
}

@media (max-width: 600px) {
  .pec-card {
    flex-direction: column;
  }

  .pec-grid {
    grid-template-columns: 1fr;
  }
}
</style>
