![](preview.png)


```yaml
type: customapi
url: http://YOUR_HOST/api/subscriptions/get_subscriptions.php?state=0&api_key=YOUR_API_KEY
refreshInterval: 300000
skip-json-validation: true
mappings:
  - field: subscriptions.0.name
    label: Next
    format: text
  - field: subscriptions.0.next_payment
    label: Date
    format: date
    dateStyle: medium
  - field: subscriptions.0.price
    label: Amount
    format: float
    suffix: "€"
template: |
  {{ if eq .Response.StatusCode 200 }}
    <div style="display: flex; flex-direction: column; gap: 15px; font-family: Arial, sans-serif;">
      <!-- Number of active subscriptions -->
      <div style="text-align: center;">
        <strong>Active subscriptions:</strong> {{ len (.JSON.Array "subscriptions") }}
      </div>                
      <!-- Next payment -->
      {{ $today := "2025-04-16" }}
      {{ $nextDate := "" }}
      {{ $nextName := "" }}
      {{ $nextPrice := 0.0 }}
      {{ range .JSON.Array "subscriptions" }}
        {{ $currentDate := .String "next_payment" }}
        {{ if gt $currentDate $today }}
          {{ if or (eq $nextDate "") (lt $currentDate $nextDate) }}
            {{ $nextDate = $currentDate }}
            {{ $nextName = .String "name" }}
            {{ $nextPrice = .Float "price" }}
          {{ end }}
        {{ end }}
      {{ end }}     
      <div style="display: flex; justify-content: space-around; margin-top: 10px;">
        <!-- Next subscription name -->
        <div style="text-align: center;">
          <strong>Next payment:</strong><br>
          {{ if gt (len $nextName) 15 }}
            {{ slice $nextName 0 15 }}...
          {{ else }}
            {{ $nextName }}
          {{ end }}
        </div>
        <!-- Next payment date -->
        <div style="text-align: center;">
          <strong>Date:</strong><br>
          {{ with $parsedDate := $nextDate | parseTime "2006-01-02" }}
            {{ $parsedDate.Format "02/01/2006" }}
          {{ else }}
            {{ $nextDate }}
          {{ end }}
        </div>
        <!-- Next payment price -->
        <div style="text-align: center;">
          <strong>Amount:</strong><br>
          {{ printf "%.2f" $nextPrice }}€
        </div>
      </div>
    </div>
  {{ end }}

   ```

## Lenguage&others

-  You can modify the label fields in the mappings section to suit your language or preferences.
