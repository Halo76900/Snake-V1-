import pygame
import random

pygame.init()

# Farben
WEISS = (255, 255, 255)
SCHWARZ = (0, 0, 0)
ROT = (255, 0, 0)
GRUEN = (0, 255, 0)
BLAU = (0, 80, 255)

# Display
spielfeld_breite = 300
spielfeld_höhe = 200
gesamthöhe = 400   # Platz für Buttons

display = pygame.display.set_mode((spielfeld_breite, gesamthöhe))
pygame.display.set_caption("Snake Raster-Version")

# Rastergröße = Apfelgröße = Schlangensegmentgröße
raster = 10

schlangen_geschwindigkeit = 5  # Snake ist jetzt größer → niedriger sinnvoll

uhr = pygame.time.Clock()
schrift = pygame.font.SysFont(None, 40)

# Buttons (groß + unter Spielfeld)
button_size = 70
buttons = {
    "oben": pygame.Rect(215, 420, button_size, button_size),
    "unten": pygame.Rect(215, 420 + button_size + 10, button_size, button_size),
    "links": pygame.Rect(135, 420 + button_size + 10, button_size, button_size),
    "rechts": pygame.Rect(295, 420 + button_size + 10, button_size, button_size),
}

def pfeil(text, rect):
    img = schrift.render(text, True, WEISS)
    w, h = img.get_size()
    display.blit(img, (rect.x + (rect.width - w) // 2, rect.y + (rect.height - h) // 2))

def zeichne_buttons():
    for r in buttons.values():
        pygame.draw.rect(display, BLAU, r, border_radius=10)
    pfeil("↑", buttons["oben"])
    pfeil("↓", buttons["unten"])
    pfeil("←", buttons["links"])
    pfeil("→", buttons["rechts"])

def button_klick(pos):
    for name, rect in buttons.items():
        if rect.collidepoint(pos):
            return name
    return None

def reset():
    """Setzt das Spiel sauber zurück."""
    schlangen_x = (spielfeld_breite // 2 // raster) * raster
    schlangen_y = (spielfeld_höhe // 2 // raster) * raster

    richtung_x = 0
    richtung_y = 0

    schlangen_liste = []
    schlangen_länge = 1

    futter_x = random.randrange(0, spielfeld_breite // raster) * raster
    futter_y = random.randrange(0, spielfeld_höhe // raster) * raster

    return schlangen_x, schlangen_y, richtung_x, richtung_y, schlangen_liste, schlangen_länge, futter_x, futter_y

def spiel_loop():
    schlangen_x, schlangen_y, richtung_x, richtung_y, schlangen_liste, schlangen_länge, futter_x, futter_y = reset()

    while True:

        # --- EVENT HANDLING ---
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                quit()

            # Tastatursteuerung
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_LEFT and richtung_x != raster:
                    richtung_x = -raster
                    richtung_y = 0
                if event.key == pygame.K_RIGHT and richtung_x != -raster:
                    richtung_x = raster
                    richtung_y = 0
                if event.key == pygame.K_UP and richtung_y != raster:
                    richtung_y = -raster
                    richtung_x = 0
                if event.key == pygame.K_DOWN and richtung_y != -raster:
                    richtung_y = raster
                    richtung_x = 0

            # Touch-Buttons
            if event.type == pygame.MOUSEBUTTONDOWN:
                klick = button_klick(event.pos)
                if klick == "oben" and richtung_y != raster:
                    richtung_x = 0
                    richtung_y = -raster
                if klick == "unten" and richtung_y != -raster:
                    richtung_x = 0
                    richtung_y = raster
                if klick == "links" and richtung_x != raster:
                    richtung_x = -raster
                    richtung_y = 0
                if klick == "rechts" and richtung_x != -raster:
                    richtung_x = raster
                    richtung_y = 0

        # --- POSITION UPDATEN ---
        schlangen_x += richtung_x
        schlangen_y += richtung_y

        # --- RANDKOLLISION → Reset ---
        if schlangen_x < 0 or schlangen_x >= spielfeld_breite or \
           schlangen_y < 0 or schlangen_y >= spielfeld_höhe:
            schlangen_x, schlangen_y, richtung_x, richtung_y, schlangen_liste, schlangen_länge, futter_x, futter_y = reset()

        # --- ZEICHNEN ---
        display.fill(SCHWARZ)

        # Spielfeldrand
        pygame.draw.rect(display, WEISS, (0, 0, spielfeld_breite, spielfeld_höhe), 2)

        # Futter (genau 1 Rasterfeld)
        pygame.draw.rect(display, ROT, (futter_x, futter_y, raster, raster))

        # Schlange
        schlangen_liste.append([schlangen_x, schlangen_y])
        if len(schlangen_liste) > schlangen_länge:
            del schlangen_liste[0]

        # Selbst-Kollision
        if [schlangen_x, schlangen_y] in schlangen_liste[:-1]:
            schlangen_x, schlangen_y, richtung_x, richtung_y, schlangen_liste, schlangen_länge, futter_x, futter_y = reset()

        for segment in schlangen_liste:
            pygame.draw.rect(display, GRUEN, (segment[0], segment[1], raster, raster))

        # Punktestand
        punkte = schrift.render(f"Punkte: {schlangen_länge - 1}", True, WEISS)
        display.blit(punkte, (10, spielfeld_höhe + 5))

        # Buttons zeichnen
        zeichne_buttons()

        pygame.display.update()

        # --- Futter gegessen ---
        if schlangen_x == futter_x and schlangen_y == futter_y:
            schlangen_länge += 1
            futter_x = random.randrange(0, spielfeld_breite // raster) * raster
            futter_y = random.randrange(0, spielfeld_höhe // raster) * raster

        uhr.tick(schlangen_geschwindigkeit)

spiel_loop()
