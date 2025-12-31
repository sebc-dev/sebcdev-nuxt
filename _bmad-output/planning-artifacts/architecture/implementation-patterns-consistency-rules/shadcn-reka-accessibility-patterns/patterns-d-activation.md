# Patterns d'Activation

## Tabs : automatic vs manual

```vue
<!-- AUTOMATIC (défaut) : active au focus -->
<Tabs activation-mode="automatic">
  <!-- Navigation rapide : flèches changent directement l'onglet -->
</Tabs>

<!-- MANUAL : requiert Enter/Space après focus -->
<Tabs activation-mode="manual">
  <!-- Meilleur si le contenu est lourd à charger -->
</Tabs>
```

## Accordion : single vs multiple

```vue
<!-- SINGLE avec collapsible : un seul ouvert, peut tout fermer -->
<Accordion type="single" collapsible>
  <AccordionItem value="faq-1">...</AccordionItem>
</Accordion>

<!-- MULTIPLE : plusieurs sections ouvertes simultanément -->
<Accordion type="multiple">
  <AccordionItem value="faq-1">...</AccordionItem>
</Accordion>
```

## Props pour contenu persistant

```vue
<!-- Garde le contenu dans le DOM même fermé (utile pour Ctrl+F) -->
<AccordionContent unmount-on-hide="false">
  Contenu searchable par le navigateur
</AccordionContent>
```
