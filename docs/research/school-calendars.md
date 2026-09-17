# French school calendars: sources and interpretation

Research for [Source of French school calendars and institution-specific closures](https://github.com/leokun/foylo/issues/4), supporting the [Foylo V1 specification map](https://github.com/leokun/foylo/issues/1). Checked September 17, 2026 against official sources. This document records evidence and proposals, not product decisions.

## Findings

The Ministry's school-calendar dataset is a suitable candidate for import, with legal calendars as the reference for interpretation. It cannot by itself determine whether a particular lunch, daycare or nanny activity is open. Source selection, supported territories and precedence remain to validate under [the domain model](../02-domain-model.md).

### Sources and coverage

| Need | Official source | Verified capability or limitation |
| --- | --- | --- |
| Legal school dates | [2026-2027 arrêté](https://www.legifrance.gouv.fr/jorf/id/JORFTEXT000052416058) and [2027-2028 arrêté](https://www.legifrance.gouv.fr/jorf/id/JORFTEXT000054457294) | Dates, academy-to-zone mapping and national bridge days; separate provisions for Corsica and overseas territories. |
| Machine-readable holidays | [Ministry dataset](https://data.education.gouv.fr/explore/dataset/fr-en-calendrier-scolaire/information/) and [data.gouv.fr resource listing](https://www.data.gouv.fr/datasets/le-calendrier-scolaire) | Records API, exports and per-zone ICS links; Licence Ouverte 2.0. |
| Municipality lookup | [Administrative geography API](https://geo.api.gouv.fr/decoupage-administratif/communes) | Municipality lookup by INSEE code and searches by name, postal code or coordinates. A postal-code search can return several municipalities. |
| Academy lookup | [Code de l'éducation R222-2](https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000042666480) | Department-to-academy mapping, for example Ain, Loire and Rhône to Lyon. |
| School lookup | [Education directory metadata](https://data.education.gouv.fr/api/explore/v2.1/catalog/datasets/fr-en-annuaire-education) | Verified fields include establishment identifier, municipality code, academy code and academy name. |
| Public holidays | [Jours fériés en France](https://www.data.gouv.fr/datasets/jours-feries-en-france) | Separate territorial calendars, JSON/CSV/ICS, Licence Ouverte 2.0. Local or professional holidays are not exhaustively represented. |

The school records expose `description`, `population`, `start_date`, `end_date`, `location`, `zones` and `annee_scolaire`. They identify the academy or territory as well as the zone. Preserve those distinctions during evaluation: deduplicating solely by zone could erase population or local differences.

The official resource listing provides Zone A/B/C, Corsica and overseas feeds. Their presence does not prove complete coverage for every future school year. On the check date, a [query for 2027-2028 grouped by zone](https://data.education.gouv.fr/api/explore/v2.1/catalog/datasets/fr-en-calendrier-scolaire/records?where=annee_scolaire%3D%222027-2028%22&select=zones,count(*)&group_by=zones) returned only Mayotte and Polynésie, while the metropolitan arrêté had already been published on July 23, 2026. This demonstrates a publication gap in this snapshot, not a guaranteed delay or update schedule.

The school resource listing displayed August 31, 2026 as its latest update. A freshness timestamp does not establish completeness. No dependable school-data refresh SLA was established in this research. The public-holiday [publisher repository](https://github.com/etalab/jours-feries-france-data) describes daily regeneration; its [dataset page](https://www.data.gouv.fr/datasets/jours-feries-en-france) advertises JSON/CSV coverage up to 20 years back and five years forward. These are different publication processes.

### Dates are not closure instants

The [Ministry's explanatory page](https://www.education.gouv.fr/calendrier-scolaire-toutes-les-dates-des-cours-et-des-vacances-100148) specifies that holidays begin after lessons on the stated date and lessons resume on the stated morning. For pupils without Saturday lessons, a Saturday holiday departure instead begins after Friday's lessons.

A live [Lyon, Zone A, 2026-2027 query](https://data.education.gouv.fr/api/explore/v2.1/catalog/datasets/fr-en-calendrier-scolaire/records?where=zones%3D%22Zone%20A%22%20and%20annee_scolaire%3D%222026-2027%22%20and%20location%3D%22Lyon%22) returned:

| Event | Raw start and end in UTC | Interpretation |
| --- | --- | --- |
| Toussaint | `2026-10-16T22:00:00+00:00` to `2026-11-01T23:00:00+00:00` | Encodes October 17 and November 2 at Paris midnight, across a daylight-saving change. The legal departure is after lessons, not Saturday 00:00. |
| Ascension bridge | Both `2027-05-06T22:00:00+00:00` | Encodes May 7. The 2026-2027 arrêté explicitly closes classes that Friday. |
| Summer start | Both `2027-07-02T22:00:00+00:00` | Encodes July 3. This is a start marker, not a one-day summer holiday. |

Consequences for a future importer, inferred from these records and the legal wording:

- Recover the relevant local date before interpreting a timestamp. Truncating the UTC string loses a day in these examples. The Paris encoding observed here must not be assumed for every overseas feed without checking it.
- Never treat all `start_date == end_date` records as empty intervals or single-day closures. Interpret the event type and its legal context. A summer-start marker does not supply a summer end.
- Never cancel Saturday lessons merely because a holiday record begins at Saturday midnight. A date-based closure model needs an explicit departure-day rule using the actual school timetable.
- The return date is a school day from the morning. If normalized whole-day intervals are introduced, define their exclusive end separately from the source timestamp convention.
- Reconcile summer end with the next published pupil rentrée, retaining its provenance. Do not invent an annual fixed rentrée date or substitute the teachers' rentrée.

### From a place to a calendar

A candidate mapping is municipality INSEE code -> department -> academy -> holiday zone, using the geography API, R222-2 and the applicable year's arrêté respectively. For a selected school, the education directory supplies the academy directly. The calendar uses names, so code/name reconciliation and historical changes need validation.

The [2026-2027 arrêté](https://www.legifrance.gouv.fr/jorf/id/JORFTEXT000052416058) places Lyon in Zone A, Strasbourg in Zone B and Paris in Zone C, and provides separate treatment for Corsica and the listed overseas territories. Corsica must not silently inherit A/B/C. Mapping from the household's home alone would also be an assumption: the relevant school or care activity can be elsewhere.

### Public holidays and local exceptions

Public-holiday API example: [`alsace-moselle/2027.json`](https://calendrier.api.gouv.fr/jours-feries/alsace-moselle/2027.json). Its checked response includes Good Friday on March 26 and December 26. However, [Code du travail L3134-13](https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000006902635) conditions Good Friday on the municipality having a Protestant or mixed church. The API's territorial holiday label is not evidence that every activity throughout the territory is closed. The specific school or service calendar still needs to be considered.

The national school arrêté explicitly includes the Ascension bridge. That does not establish the opening policy of daycare, school lunch or another Activity. Institution notices, municipality calendars and family/provider agreements may supply additional information. The sources checked here did not establish a comprehensive national feed for these activity-specific openings and closures. Local public feeds may exist and were not exhaustively surveyed.

## Proposals to validate

1. Import official school calendars as optional inputs to Calendar, retaining source, school year, academy/territory, population and retrieval time. School holidays need not govern daycare or nanny activities.
2. Keep public holidays distinct from school holidays and explicit institution openings/closures, then combine them with ScheduleException. The precedence between these inputs is a product decision, not something the datasets resolve.
3. Represent unavailable calendar coverage as unknown, with an explicit user-facing policy. Missing records are not evidence of an open day or a closed day. Preserve a known cached version when a refresh fails.
4. Recheck coverage when extending the planning horizon and choose a routine refresh policy after defining freshness requirements. Monthly polling is an option, not an established requirement or source guarantee.
5. Preserve past calendar versions or effective periods so an upstream correction does not silently reinterpret historical plans. The storage mechanism remains open.

## Decisions still required

- Which territories and institution types are in V1? Which calendar belongs to each Place or Activity?
- How are lessons and after-school care on the holiday departure day represented?
- Who resolves missing years, unexpected record shapes and conflicting official/local information?
- Which explicit opening or closure takes priority over school holidays, public holidays and a Person's ScheduleException?
- How are source attribution, freshness and unknown coverage shown?

Before implementation, acceptance examples should cover a taught Saturday, a Friday after-school slot before a holiday, the return morning, a daylight-saving boundary, a summer-start marker, daycare open during school holidays, missing future coverage and an institution-specific closure. No importer or product behavior was implemented by this research.
