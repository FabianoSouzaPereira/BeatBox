package Content;

public class BeatBox {
	/** Estes são os nomes dos instrumentos, como uma array de strings, para a construção dos rótulos de GUI*/
	static String[] instrumentNames = { "Bass Drum", "Closed Hi-Hat", "Open Hi-Hat",
		"Acoust Snare", "Crash", "Cymbal", "Hand Clap", "High Tom",
		"Hi Bongo", "Maracas", "Whistle", "Low Conga", "Cowbell",
		"Vibraslap", "Low-mid Tom", "High Agogo", "Open Hi Conga" };
	
	public static void main(String[] args) {
		new BeatBox2().buildGUI();
	}

}

package Content;

import java.awt.BorderLayout;
import java.awt.GridLayout;
import java.awt.Label;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.util.ArrayList;

//import Content.BeatBox;

import javax.sound.midi.MidiEvent;
import javax.sound.midi.MidiSystem;
import javax.sound.midi.Sequence;
import javax.sound.midi.Sequencer;
import javax.sound.midi.ShortMessage;
import javax.sound.midi.Track;
import javax.swing.BorderFactory;
import javax.swing.Box;
import javax.swing.BoxLayout;
import javax.swing.JButton;
import javax.swing.JCheckBox;
import javax.swing.JFrame;
import javax.swing.JPanel;

public class BeatBox2 {
	JFrame theFrame;
	ArrayList<JCheckBox> checkboxList;
	JPanel mainPanel;
	Sequencer sequencer;
	Sequence sequence;
	Track track;
	
	/** Esses múmeros representam as 'teclas' reais da bateria.*/
	int[] instruments = { 35, 42, 46, 38, 49, 39, 50, 60, 70, 72, 64, 56, 58,
			47, 67, 63 };
	
	public void buildGUI() {
		theFrame = new JFrame("Cyber BeatBox");
		theFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		BorderLayout layout = new BorderLayout();
		JPanel background = new JPanel(layout);
		background.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10));

		checkboxList = new ArrayList<JCheckBox>();
		Box buttonBox = new Box(BoxLayout.Y_AXIS);
		
		JButton start = new JButton("Start");
		start.addActionListener(new MyStartListener());
		buttonBox.add(start);
		
		JButton stop = new JButton("Stop");
		stop.addActionListener(new MyStopListener());
		buttonBox.add(stop);
		
		JButton upTempo = new JButton("Tempo Up");
		upTempo.addActionListener(new MyUpTempoListener());
		buttonBox.add(upTempo);
		
		JButton downTempo = new JButton("Tempo Down");
		downTempo.addActionListener(new MyDownTempoListener());
		buttonBox.add(downTempo);
		
		Box nameBox = new Box(BoxLayout.Y_AXIS);
		for (int i = 0; i < 16; i++) {
			nameBox.add(new Label(Content.BeatBox.instrumentNames[i]));
		}
		
		background.add(BorderLayout.EAST, buttonBox);
		background.add(BorderLayout.WEST, nameBox);
		
		theFrame.getContentPane().add(background);
		
		GridLayout grid = new GridLayout(16,16);
		grid.setVgap(1);
		grid.setHgap(2);
		mainPanel = new JPanel(grid);
		background.add(BorderLayout.CENTER, mainPanel);
		
		for (int i = 0; i < 256; i++){
			JCheckBox c = new JCheckBox();
			c.setSelected(false);
			checkboxList.add(c);
			mainPanel.add(c);
			//fim do loop
		}
		
		setUpMidi();
		
		theFrame.setBounds(50, 50, 300, 300);
		theFrame.pack();
		theFrame.setVisible(true);
		//fecha o metodo
	}

	/**
	 * O costumeiro trecho de configuração MIDI para a captura do sequenciador, 
	 * da sequencia e da faixa. Novamente, nada de especial.
	 * @exception e.printStackTrace();
	 */
	private void setUpMidi() {
		try{
			sequencer = MidiSystem.getSequencer();
			sequencer.open();
			sequence = new Sequence(Sequence.PPQ,4);
			track = sequence.createTrack();
			sequencer.setTempoInBPM(120);
			
		} catch(Exception e) {
			e.printStackTrace();
		}  //fecha o metodo
		
	}
	
	/**
	 * É aqui que tudo acontece! é o local em que convertemos o estado da caixa 
	 * de seleção em eventos MIDI  e os adicionamos à Faixa.
	 */
	public void buildTrackAndStart(){
		int[] trackList = null;
		
		sequence.deleteTrack(track);		//Elimina a faixa antiga
		track = sequence.createTrack();	//Cria uma faixa nova
		
		for(int i = 0; i < 16; i++){    //fará isso para 
			trackList = new int[16];	//todas as 16 linhas
			
			int key = instruments[i];  //configura a tecla que representara qual 
									  //é esse instrumento
			
			for(int j = 0; j < 16; j++){
				
				JCheckBox jc =(JCheckBox) checkboxList.get(j + (16 * i));
				if (jc.isSelected()){
					trackList[j] = key;
				} else {
					trackList[j] = 0;
				}
				
			} //fecha o looping interno
			
	/**Para esse instrumento, e para todas as 16 batisdas, cria eventos e adiciona à faixa. */		
			makeTracks(trackList);
			track.add(makeEvent(176,1,127,0,16));
		} //fecha o looping externo
		
/**Queremos no certificar que sempre de que há um evento na batida 16(ela vai de 0 a 15).
*  Caso contraio , a BetBox pode não percorrer todas as 16 batidas antes de começar novamente*/
		
		track.add(makeEvent(192,9,1,0,15));
		try{
			
			sequencer.setSequence(sequence);
			sequencer.setLoopCount(Sequencer.LOOP_CONTINUOUSLY);
			sequencer.start();
			sequencer.setTempoInBPM(120);		
			
		} catch(Exception e) {e.printStackTrace();}
	}
	
	public class MyStartListener implements ActionListener{
		public void actionPerformed(ActionEvent a){
			buildTrackAndStart();
		}
	} //fecha a classe interna
	
	public class MyStopListener implements ActionListener{
		public void actionPerformed(ActionEvent a){
			sequencer.stop();
		}
	} //fecha a classe interna
	
	public class MyUpTempoListener implements ActionListener{
		public void actionPerformed(ActionEvent a){
			float tempoFactor = sequencer.getTempoFactor();
			sequencer.setTempoFactor((float)(tempoFactor * 1.03));
		}
	} //fecha a classe interna
	
	public class MyDownTempoListener implements ActionListener{
		public void actionPerformed(ActionEvent a){
			float tempoFactor = sequencer.getTempoFactor();
			sequencer.setTempoFactor((float)(tempoFactor * .97));
		}
	} //fecha a classe interna
	
	
	
	private void makeTracks(int[] list) {

		for (int i = 0; i < 16; i++){
			int key = list[i];
			
			if (key != 0){
				track.add(makeEvent(144,9, key, 100, i));   //cria os efentos note on
				track.add(makeEvent(128,9, key, 100, i+1));  //e note off e os adiciona a faixa
			}
		}
		
	}
	
	private MidiEvent makeEvent(int comd, int chan, int one, int two, int tick) {
		MidiEvent event = null;
		try {
			ShortMessage a = new ShortMessage();
			a.setMessage(comd, chan, one, two);
			event = new MidiEvent(a, tick);
			
		} catch (Exception e) {e.printStackTrace();	}
	return event;
	}

	
}
