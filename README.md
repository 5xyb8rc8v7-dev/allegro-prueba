#include <allegro5/allegro.h>
#include <allegro5/allegro_primitives.h>
#include <math.h>
int main() {
   // Inicialización básica de Allegro
   al_init();
   al_init_primitives_addon();
   // Crear la ventana (Display) de 600x600
   ALLEGRO_DISPLAY *display = al_create_display(600, 600);
   ALLEGRO_EVENT_QUEUE *queue = al_create_event_queue();
   ALLEGRO_TIMER *timer = al_create_timer(1.0 / 60.0); // 60 FPS
   // Registrar eventos para poder cerrar la ventana
   al_register_event_source(queue, al_get_display_event_source(display));
   al_register_event_source(queue, al_get_timer_event_source(timer));
   al_start_timer(timer);
   // Variables del radar
   float cx = 300, cy = 300; // Centro de la pantalla
   float radius = 250;       // Radio del radar
   float angle = 0;          // Ángulo actual de la sombra
   // Variables del objetivo (target)
   float target_angle = 5.0; // Ángulo donde está el objetivo
   float target_dist = 150;  // Distancia desde el centro
   // Coordenadas cartesianas del objetivo
   float tx = cx + target_dist * cos(target_angle);
   float ty = cy + target_dist * sin(target_angle);
   bool running = true;
   // Ciclo principal del programa
   while(running) {
       ALLEGRO_EVENT event;
       al_wait_for_event(queue, &event);
       // Si se cierra la ventana, salir del ciclo
       if(event.type == ALLEGRO_EVENT_DISPLAY_CLOSE) {
           running = false;
       }
       // Actualización y dibujo (60 veces por segundo)
       if(event.type == ALLEGRO_EVENT_TIMER) {
           // Aumentar el ángulo para que el radar gire
           angle += 0.03;
           // Mantener el ángulo entre 0 y 2*PI
           if(angle >= ALLEGRO_PI * 2) angle -= ALLEGRO_PI * 2;
           // 1. Limpiar pantalla en negro
           al_clear_to_color(al_map_rgb(0, 0, 0));
           // 2. Dibujar contorno del radar (verde oscuro)
           al_draw_circle(cx, cy, radius, al_map_rgb(0, 150, 0), 2);
           // 3. Dibujar la "sombra" (un octavo de círculo = PI/4)
           // Se usa color verde con transparencia simulada
           al_draw_filled_pieslice(cx, cy, radius, angle, ALLEGRO_PI / 4, al_map_rgb(0, 80, 0));
           // 4. Detectar si la sombra pasa sobre el objetivo
           // Calculamos la diferencia entre el ángulo del radar y el del objetivo
           float diff = target_angle - angle;
           // Normalizar la diferencia
           if(diff < 0) diff += ALLEGRO_PI * 2;
           // Si la diferencia es menor al ancho de la sombra (PI/4), hay contacto
           if(diff >= 0 && diff <= ALLEGRO_PI / 4) {
               // Enfatizar objetivo: Rojo y más grande
               al_draw_filled_circle(tx, ty, 8, al_map_rgb(255, 0, 0));
           } else {
               // Objetivo normal: Verde oscuro y pequeño
               al_draw_filled_circle(tx, ty, 4, al_map_rgb(0, 200, 0));
           }
           // Mostrar los cambios en pantalla
           al_flip_display();
       }
   }
   // Liberar memoria
   al_destroy_timer(timer);
   al_destroy_event_queue(queue);
   al_destroy_display(display);
   return 0;
}
