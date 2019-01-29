# Littlevgl library working on mbed

Derived from https://github.com/littlevgl/lvgl with minor changes, tested on F469-Discovery board.

Should work with screen drivers: [TFT](https://github.com/stepansnigirev/f469-disco-lvgl-tft) and [Touchpad](https://github.com/stepansnigirev/f469-disco-lvgl-touchpad)

Docs are available on [Littlevgl website](https://docs.littlevgl.com/)

Simple example:

```cpp
#include <mbed.h>
#include <tft.h>
#include <touchpad.h>
#include <lvgl.h>

/* timer to count time in a loop and update the lvgl */
volatile int t = 0;
Ticker ms_tick;
void onMillisecondTicker(void){
  t++;
}

/* global label to set click count */
static lv_obj_t * lbl;
/* click counter */
int cnt = 0;

/* button callback */
static lv_res_t btn_click_action(lv_obj_t * btn){
  char msg[40];
  cnt++;
  sprintf(msg, "button clicked %d times", cnt);
  lv_label_set_text(lbl, msg);
  return LV_RES_OK; /*Return OK if the button is not deleted*/
}

int main() {

  ms_tick.attach_us(onMillisecondTicker, 1000);

  /* init hardware */
  lv_init();
  tft_init();
  touchpad_init();

  /* define theme */
  lv_theme_t * th = lv_theme_material_init(210, NULL);
  lv_theme_set_current(th);

  /* create and load screen */
  lv_obj_t * scr = lv_cont_create(NULL, NULL);
  lv_scr_load(scr);

  /* create a label to log clicks */
  lbl = lv_label_create(lv_scr_act(), NULL);
  lv_label_set_text(lbl, "Hello lvgl!");
  lv_label_set_long_mode(lbl, LV_LABEL_LONG_BREAK);
  lv_label_set_align(lbl, LV_LABEL_ALIGN_CENTER);
  lv_obj_set_width(lbl, 450);
  lv_obj_align(lbl, NULL, LV_ALIGN_IN_TOP_MID, 0, 200);

  /* Create simple button */
  lv_obj_t * btn = lv_btn_create(lv_scr_act(), NULL);
  lv_obj_set_width(btn, 300);
  lv_obj_set_height(btn, 100);
  lv_obj_align(btn, lbl, LV_ALIGN_OUT_BOTTOM_MID, 0, 100);
  lv_btn_set_action(btn, LV_BTN_ACTION_CLICK, btn_click_action);

  /* Add a label to the button*/
  lv_obj_t * label = lv_label_create(btn, NULL);
  lv_label_set_text(label, "Click me!");

  while(1) {
    HAL_Delay(1);
    lv_tick_inc(t);
    lv_task_handler();
    t = 0;
  }
}
```