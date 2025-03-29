![](preview.png)


```yaml
- type: custom-api
  title: NextDNS Analytics
  cache: 1h
  url: https://api.nextdns.io/profiles/${NEXTDNS_PROFILE_ID}/analytics/status
  headers:
    X-Api-Key: ${NEXTDNS_API_KEY}
  template: |
    {{ if eq .Response.StatusCode 200 }}
      <div style="display: flex; justify-content: space-between;">
        {{ $total := 0.0 }}
        {{ $blocked := 0.0 }}
        
        {{ range .JSON.Array "data" }}
          {{ $total = add $total (.Int "queries" | toFloat) }}
          {{ if eq (.String "status") "blocked" }}
            {{ $blocked = add $blocked (.Int "queries" | toFloat) }}
          {{ end }}
        {{ end }}
        
        <div style="flex: 1; text-align: center;">
          <p>Queries</p>
          <p>{{ printf "%.0f" $total }}</p>
        </div>
        <div style="flex: 1; text-align: center; color: var(--color-negative);">
          <p>Blocked</p>
          <p>{{ printf "%.0f" $blocked }}</p>
        </div>
        <div style="flex: 1; text-align: center;">
          <p>Block Rate</p>
          {{ if gt $total 0.0 }}
            <p>{{ div (mul $blocked 100) $total | printf "%.2f" }}%</p>
          {{ else }}
            <p>0.00%</p>
          {{ end }}
        </div>
      </div>
    {{ else }}
      <div style="text-align: center; color: var(--color-negative);">
        Error: {{ .Response.StatusCode }} - {{ .Response.Status }}
      </div>
    {{ end }}
   ```

## Environment variables

- `GHOSTFOLIO_URL` - the URL of the Ghostfolio instance
- `GHOSTFOLIO_PUBLIC_URL` - You need to create a public acces url under menu My Ghostfolio -> Acces - Add
