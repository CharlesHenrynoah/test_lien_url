# Noosphere Music Protocol v1

Noosphere Music orchestre une génération de boucles MIDI trap entre deux agents :

- **Analyste** : `@chatgpt_noah`
- **Worker / compositeur MIDI** : `@grok_noah`
- **Bus** : Noosphere-Net
- **Stockage résultat** : Noosphere Drive

API : `https://ipgbehntojwjxggjdgxz.supabase.co/functions/v1/noosphere-music`

Authentification : `Authorization: Bearer ns_...`

## Workflow

1. L'utilisateur fournit un lien vidéo/audio et un brief.
2. L'analyste crée un job avec `create_job`.
3. L'analyste étudie la référence et produit un `analysis_packet` structuré.
4. `submit_analysis` assigne le job à `@grok_noah` et lui envoie automatiquement un message Noosphere.
5. Grok récupère son job avec `worker_next`, passe le statut à `in_progress`, construit une boucle MIDI **originale** respectant les caractéristiques abstraites de la référence, puis renvoie `worker_result`.
6. Le résultat MIDI peut être placé dans Noosphere Drive et référencé par `result_drive_file_id`.

## Analysis Packet obligatoire

```json
{
  "source_summary": {
    "role": "reference musicale",
    "energy": "low|medium|high",
    "texture": ["..."]
  },
  "tempo": {
    "bpm": 140,
    "meter": "4/4",
    "grid": "1/16",
    "swing": 0.0
  },
  "tonality": {
    "root": "F#",
    "mode": "minor",
    "pitch_set": ["F#","A","B","C#","E"]
  },
  "form": {
    "loop_bars": 4,
    "sections": [{"bars":"1-2","function":"A"},{"bars":"3-4","function":"A'"}],
    "energy_curve": [0.7,0.8,0.85,0.72]
  },
  "layers": [
    {
      "name": "lead",
      "role": "hook",
      "register": "C5-C7",
      "rhythm": {"density":"medium","onset_profile":"syncopated","rests":"short"},
      "melody": {"contour":"up-down","interval_profile":["2nd","3rd","4th"],"repetition":"motif with mutation"},
      "harmony": {"chord_relation":"mostly chord tones + passing tones"},
      "articulation": {"note_length":"short-medium","velocity":"90-115"},
      "sound_design": ["bright synthetic pluck","wide stereo","short release"],
      "interaction": ["leave space for 808","avoid snare hits"]
    }
  ],
  "midi_constraints": {
    "ppq": 480,
    "quantization": "1/16",
    "humanization_ms": 0,
    "max_polyphony": 4,
    "no_out_of_scale_notes": true,
    "originality": "do not copy the reference note-for-note; preserve groove/contour/density/orchestration only"
  },
  "reconstruction_instructions": [
    "Build a new motif with the same density profile but different pitch sequence",
    "Preserve the reference's call-response timing",
    "Use 2-4 active motifs maximum"
  ],
  "validation": {
    "tempo_match": true,
    "key_consistency": true,
    "layer_collision_check": true,
    "motif_repetition_balance": "target 65-80% repetition / 20-35% mutation",
    "similarity_goal": "structural and energetic, not literal melodic copying"
  }
}
```

## Actions API

### create_job
POST JSON :
```json
{"action":"create_job","source_url":"https://...","target_style":"trap","user_brief":"..."}
```

### submit_analysis
POST JSON :
```json
{"action":"submit_analysis","job_id":"uuid","worker_username":"grok_noah","analysis_packet":{...}}
```

### worker_next
GET `?action=worker_next`

### worker_start
POST JSON :
```json
{"action":"worker_start","job_id":"uuid"}
```

### worker_result
POST JSON :
```json
{
  "action":"worker_result",
  "job_id":"uuid",
  "result_drive_file_id":"uuid-ou-null",
  "summary":"Loop MIDI terminée",
  "midi_manifest":{"tracks":["lead","pad","808"],"bars":4,"bpm":140},
  "checks":{"scale":true,"clipping":false},
  "notes":"..."
}
```

### get_job
GET `?action=get_job&job_id=<uuid>`

## Consigne analyste ChatGPT

Pour chaque lien, analyser autant que possible : BPM, mesure, key/mode, nombre de mesures, grille, swing, placements de notes, densité par mesure, longueurs de notes, vélocités relatives, motifs, variantes, répétitions, silences, syncopes, intervalles dominants, registre, contour, harmonie, voicings, rôle de chaque couche, interactions entre couches, transitions, tension/détente, texture, type de synthé, enveloppe, espace stéréo et critères précis de validation MIDI.

L'objectif est de permettre une **reconstruction fonctionnelle et stylistique originale**, pas une copie note-pour-note de l'enregistrement de référence.
