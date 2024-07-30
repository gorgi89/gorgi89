# main.py
import kivy
from kivy.app import App
from kivy.uix.label import Label
from kivy.uix.boxlayout import BoxLayout
from kivy.clock import Clock
from kivy.uix.popup import Popup

import requests
from bs4 import BeautifulSoup
from plyer import notification

kivy.require('2.0.0')

class LiveScoreApp(App):
    def build(self):
        self.layout = BoxLayout(orientation='vertical')
        self.label = Label(text='Fetching live scores...')
        self.layout.add_widget(self.label)
        
        Clock.schedule_interval(self.update_scores, 60)  # Update every 60 seconds
        return self.layout

    def update_scores(self, dt):
        try:
            response = requests.get('https://www.rezultati.com/')
            soup = BeautifulSoup(response.text, 'html.parser')
            # Note: Actual scraping logic needs to be implemented based on the site structure
            scores = self.parse_scores(soup)
            self.label.text = '\n'.join(scores)
            self.check_for_goals(scores)
        except Exception as e:
            self.label.text = 'Error fetching scores'

    def parse_scores(self, soup):
        # Placeholder implementation
        matches = soup.find_all('div', class_='event__match')
        scores = []
        for match in matches:
            home_team = match.find('div', class_='event__participant--home').text
            away_team = match.find('div', class_='event__participant--away').text
            home_score = match.find('div', class_='event__score--home').text
            away_score = match.find('div', class_='event__score--away').text
            scores.append(f"{home_team} {home_score} - {away_team} {away_score}")
        return scores

    def check_for_goals(self, scores):
        # Placeholder logic for detecting new goals
        # This should be replaced with actual state management and comparison with previous state
        for score in scores:
            if '1 - 0' in score:  # Example condition
                self.show_notification(score)
                self.show_popup(score)

    def show_notification(self, message):
        notification.notify(
            title='Goal Scored!',
            message=message,
            app_name='Live Score App',
            timeout=10
        )

    def show_popup(self, message):
        popup = Popup(title='Goal Scored!',
                      content=Label(text=message),
                      size_hint=(0.5, 0.5))
        popup.open()

if __name__ == '__main__':
    LiveScoreApp().run()
