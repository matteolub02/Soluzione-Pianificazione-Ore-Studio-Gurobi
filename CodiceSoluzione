package soluzione;

import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;
import java.util.Scanner;
import gurobi.*;

public class Soluzione_instance24 {
	
	private static final String COGNOME ="LUBLANIS";
	private static final String GRUPPO = "instance-24";
	private static String soluzioneAVideo = ("\nGRUPPO: " + GRUPPO + "\nComponenti: " + COGNOME + "\n\nQUESITO I:\n");
	private static int d = 0, l = 0, tmax = 0, 
			k = 0, a = 0, b = 0, c = 0;
	private static ArrayList<Integer> t = new ArrayList<>(); //time needed for subject i 
	private static ArrayList<Integer> tau = new ArrayList<>(); //time needed if a subject is selectioned
	
	public static void main (String[] args) throws IOException {
		setupData(); //setup data from .txt file
		GRBEnv env; 
		try {
			env = new GRBEnv("instance-24.log");  
			setParams(env); 
			
			GRBModel model = new GRBModel(env); 
			
			
			//QUESITO I
			GRBVar[][] x_ij = addVarsTime(model); //time subject i day j
			GRBVar[][] y_ij = addVarsBinary(model); //binary selection subject i day j
			addFO(model, x_ij); 
			addConstr(model, x_ij, y_ij); 
			model.update();
			model.optimize(); 
			soluzioneAVideo += firstString(model); //FIRST PART - FIRST QUESTION	
			GRBModel rilassato = model.relax(); 
			rilassato.update();
			rilassato.optimize(); 
			/*
			 * When I relax this model, time and binary variables become continuous.
			 * Since the binary type was fundamental for the constraints three and four to work, 
			 * we now have 13 non-binding constraints, as it can change however it wants. 
			 * If we also check the relaxed solution, we can find that on day 12 (with the instance in my possession)
			 * you can study three subjects, which should not be possible with the initial conditions.
			 */
			//soluzioneAVideo += firstString(rilassato);
			soluzioneAVideo += secondString(rilassato); //SECOND PART - FIRST QUESTION
			isDegenerative(rilassato);
			multipleSol(rilassato);
			relaxedEqualsInteger(model, rilassato); //THIRD PART - FIRST QUESTION
			
			//QUESITO II
			addConstr5(model, y_ij);
			addConstr6(model, y_ij);
			model.update();
			model.optimize(); 
			soluzioneAVideo += "\nQUESITO II:\n";
			//check if the solution is correct, removed from the solution since it's not required
			//soluzioneAVideo += firstString(model);
			//checkSolutionSecondQuestion(x_ij); 
			soluzioneAVideo += "Obj nuovo intero: " 
					+ model.get(GRB.DoubleAttr.ObjVal) + "\n"; 
			
			//QUESITO III
			soluzioneAVideo += "\nQUESITO III:\n"; 
			sensitivitaRelaxed(rilassato);
			minTmax(env, model, x_ij); 
			
			//PRINT SOLUTION
		    System.out.println(soluzioneAVideo); 
			dispose(model, rilassato, env);
			
			//SAVE SOLUTION IN A .TXT FILE
			try (FileWriter myWriter = new FileWriter(new File("soluzione.txt"))) {
				myWriter.write(soluzioneAVideo);
				myWriter.close();
			}
			catch (Exception e) {
				System.out.println("Preghiamo che tra tutto non sia FileWriter a non funzionare.");
			}
		} catch (GRBException e) {
			System.out.println("Gurobi Exception");
			e.printStackTrace();
		}
	}
	
	@SuppressWarnings("unused")
	private static void checkSolutionSecondQuestion(GRBVar[][] x) throws GRBException {
		for (int i = 0; i < d; i++) {
			System.out.println("GIORNO " + (i + 1));
			System.out.println("A: " + x[a][i].get(GRB.DoubleAttr.X));
			System.out.println("B: " + x[b][i].get(GRB.DoubleAttr.X));
			System.out.println("C: " + x[c][i].get(GRB.DoubleAttr.X));
		}
	}

	public static void setupData () 
	{
		File data = new File(GRUPPO + ".txt"); 
		try (Scanner sc = new Scanner(data)) {
			while (sc.hasNextLine()) {
				String[] line = sc.nextLine().split("\\s+");
				switch (line[0]) {
				case "d":
					d = Integer.parseInt(line[1].replaceAll("\\s", ""));
					break;
				case "t":
					for (int i = 1; i < line.length; i++) 
						t.add(Integer.parseInt(line[i].replaceAll("\\s", "")));
					break;
				case "l":
					l = Integer.parseInt(line[1].replaceAll("\\s", ""));
					break;
				case "tau":
					for (int i = 1; i < line.length; i++) 
						tau.add(Integer.parseInt(line[i].replaceAll("\\s", "")));
					break;
				case "tmax":
					tmax = Integer.parseInt(line[1].replaceAll("\\s", ""));
					break;
				case "k":
					k = Integer.parseInt(line[1].replaceAll("\\s", ""));
					break;
				case "a":
					a = Integer.parseInt(line[1].replaceAll("\\s", ""));
					break;
				case "b":
					b = Integer.parseInt(line[1].replaceAll("\\s", ""));
					break;
				case "c":
					c = Integer.parseInt(line[1].replaceAll("\\s", ""));
					break;
				}			
			}
		} catch (FileNotFoundException e) {
			System.out.println("File not found.");
			e.printStackTrace();
		} catch (NumberFormatException e) {
			e.printStackTrace();
		}
	}
	
	//setup gurobi params
	private static void setParams(GRBEnv env) throws GRBException 
	{
		env.set(GRB.IntParam.Method, 0);
		env.set(GRB.IntParam.Presolve, 0);
		env.set(GRB.DoubleParam.Heuristics, 0);
	}

	/*
	 * QUESITO I
	 */
	
	//add time variables
	private static GRBVar[][] addVarsTime(GRBModel model) throws GRBException 
	{
		GRBVar x_id[][] = new GRBVar[t.size()][d]; //x[i][d] ove i indica materie d giorni
		for(int i = 0; i < t.size(); i++) //materie
			for(int j = 0; j < d; j++) //giorni
				x_id[i][j]= model.addVar(0, GRB.INFINITY, 1, GRB.INTEGER, "x_" + i + "_" + j); //hours number
		return x_id;
	}
	
	//add binary variables
	private static GRBVar[][] addVarsBinary(GRBModel model) throws GRBException
	{
		GRBVar y_id[][] = new GRBVar[t.size()][d];
		for(int i = 0; i < t.size(); i++) //materie
			for(int j = 0; j < d; j++) //giorni
				y_id[i][j]= model.addVar(0, 1, 1, GRB.BINARY, "y_" + i + "_" + j); //binary selection
		
		
		return y_id;
	}
	
	//fo = max sum_i (x(k,i)) 
	private static void addFO(GRBModel model, GRBVar[][] vars) throws GRBException 
	{
		GRBLinExpr expr = new GRBLinExpr();
		for (int i = 0; i < d; i++) expr.addTerm(1, vars[k][i]);
		
		model.setObjective(expr, GRB.MAXIMIZE);
		
		
		/* TODO rimuovi questo
		model.set(GRB.IntParam.PoolSearchMode, 2);
		model.set(GRB.IntParam.PoolSolutions, 10);
		model.set(GRB.IntParam.MIPFocus, 2); 
		*/
	}
	
	
	private static void addConstr(GRBModel model, GRBVar[][] x, GRBVar[][] y) throws GRBException 
	{
		addConstrLSubjectsADay(model, y);
		addConstr1(model, x); 
		addConstr2(model, x);
		addConstr3(model, x, y); 
		addConstr4(model, x, y);
	}
	
	//max l subjects a day
	private static void addConstrLSubjectsADay (GRBModel model, GRBVar[][] y) throws GRBException {
		for(int j = 0; j < d; j++) {
			GRBLinExpr lhs = new GRBLinExpr();

			for (int i = 0; i < t.size(); i++) {
				lhs.addTerm(1, y[i][j]);
			}
			model.addConstr(lhs, GRB.LESS_EQUAL, l, "max_materie_giorno_" + j);  
		}
	}
	//study at least t_i hours in d days for subject i
	private static void addConstr1(GRBModel model, GRBVar[][] x) throws GRBException 
	{
		for(int i = 0; i < t.size(); i++) {
			GRBLinExpr lhs = new GRBLinExpr();
			for (int j = 0; j < d; j++) {
				lhs.addTerm(1, x[i][j]);
			}
			model.addConstr(lhs, GRB.GREATER_EQUAL, t.get(i), "min_ore_dgiorni_materia_" + i); 
		}
	}
	
	//study for a max of tmax hours a day
	private static void addConstr2 (GRBModel model, GRBVar[][] x) throws GRBException 
	{
		for(int j = 0; j < d; j++) {
			GRBLinExpr lhs = new GRBLinExpr();

			for (int i = 0; i < t.size(); i++) {
				lhs.addTerm(1, x[i][j]);
			}
			model.addConstr(lhs, GRB.LESS_EQUAL, tmax, "max_ore_giorno_" + j); 
		}
		
	}
	
	//min tau hours for subject i, y says if such subj is selected or not 
	private static void addConstr3(GRBModel model, GRBVar[][] x, GRBVar[][] y) throws GRBException 
	{		
		for(int i = 0; i < t.size(); i++) {

			for (int j = 0; j < d; j++) {
				GRBLinExpr lhs = new GRBLinExpr();
				lhs.addTerm(1, x[i][j]);
				lhs.addTerm(-tau.get(i), y[i][j]);
				model.addConstr(lhs, GRB.GREATER_EQUAL, 0, "min_ore_materia_" + i + "_giorno_" + j); 		
			}
		}
	}

	//max hours for subject i, y says if such subj is selected or not
	private static void addConstr4 (GRBModel model, GRBVar[][] x, GRBVar[][] y) throws GRBException
	{
		for (int i = 0; i < t.size(); i++) {
			for (int j = 0; j < d; j++) {
				GRBLinExpr lhs = new GRBLinExpr();
				lhs.addTerm(1, x[i][j]);
				lhs.addTerm(-tmax, y[i][j]);
				model.addConstr(lhs, GRB.LESS_EQUAL, 0, "max_ore_materia_" + i + "_giorno_" + j); 
			}
		}
	}
	
	//check if there are multiple solutions
	private static void multipleSol(GRBModel model) throws GRBException
	{
		for(GRBVar v: model.getVars()) {
			if (v.get(GRB.IntAttr.VBasis) != 0) { //check if var isn't basic
				if (v.get(GRB.DoubleAttr.RC) >= 0) { //check rc
					if (v.get(GRB.DoubleAttr.RC) < 1e-6) {
						soluzioneAVideo += "Multipla: si\n"; //rc = 0, so it's not unique
						return;
					}
 
				}
				else if (v.get(GRB.DoubleAttr.RC) > -(1e-6)) {
					soluzioneAVideo += "Multipla: si\n"; //rc = 0, so it's not unique
					return;
				}
			}
		}
		
		for(GRBConstr v: model.getConstrs()) {
			if (v.get(GRB.IntAttr.CBasis) != 0) { //check for slack variables
				if (v.get(GRB.DoubleAttr.Pi) >= 0) { 
					if (v.get(GRB.DoubleAttr.Pi) < 1e-6) {
						soluzioneAVideo += "Multipla: si\n"; 
						return;
					}
				}
				else if (v.get(GRB.DoubleAttr.Pi) > -(1e-6)) {
					soluzioneAVideo += "Multipla: si\n"; 
					return;
				}
			}
		}
		
		soluzioneAVideo += "Multipla: no\n"; //no rc equals 0, so it's unique
		return;
	}
	
	//check if the solution is degenerative
	private static void isDegenerative (GRBModel model) throws GRBException {
		
		for(GRBVar v: model.getVars()) {
			if (v.get(GRB.IntAttr.VBasis) == 0) { //check if the var is basic
				if (v.get(GRB.DoubleAttr.X) >= 0) { //check if the var has 0 value
					if (v.get(GRB.DoubleAttr.X) < 1e-6) {
						soluzioneAVideo += "Degenere: si\n";
						return;
					}
				}
				else if (v.get(GRB.DoubleAttr.X) > -(1e-6)) {
					soluzioneAVideo += "Degenere: si\n";
					return;
				}
			}
		}
		
		for(GRBConstr v: model.getConstrs()) {
			if (v.get(GRB.IntAttr.CBasis) == 0) { //check - slack variables
				if (v.get(GRB.DoubleAttr.Slack) >= 0) { 
					if (v.get(GRB.DoubleAttr.Slack) < 1e-6) {
						soluzioneAVideo += "Degenere: si\n";
						return;
					}

				}
				else if (v.get(GRB.DoubleAttr.Slack) > -(1e-6)) {
					soluzioneAVideo += "Degenere: si\n";
					return;
				}
			}
		}
		soluzioneAVideo += "Degenere: no\n"; //no basic variable equals zero, so it's not degenerative
		return;

	}
	//check if the solution found for the relaxed model is the same as the first model's solution
	private static void relaxedEqualsInteger (GRBModel model, GRBModel rilassato) throws GRBException 
	{
		/*
		 * if the variables of the relaxed model are all integers, then the solutions are the same
		 */
		for (GRBVar v : rilassato.getVars()) {

			if (v.get(GRB.DoubleAttr.X) != (int)v.get(GRB.DoubleAttr.X)) { //if var is not an integer
	            soluzioneAVideo += "Rilassato coincide con Intero: no\n";  //then they don't correspond
	            return;
			}
		}
		
		if (rilassato.get(GRB.DoubleAttr.ObjVal) == model.get(GRB.DoubleAttr.ObjVal)) { 
            soluzioneAVideo += "Rilassato coincide con Intero: si\n"; 
            return;
		}
		else {
			soluzioneAVideo += "Rilassato coincide con Intero: no\n"; 
			return;
		}
		
	}
	//returns string for the first request
	private static String firstString(GRBModel model) throws GRBException 
	{	
		
		String s = "";
		s += "Obj intero: " + model.get(GRB.DoubleAttr.ObjVal) + "\n"; 
		s += ("VALORE VARIABILI:\n");
		for(GRBVar v: model.getVars()) {
			/*if (v.get(GRB.DoubleAttr.X) > 0)*/ 
				s +=(v.get(GRB.StringAttr.VarName) + " = " + v.get(GRB.DoubleAttr.X)) + "\n";
		}
		for (GRBConstr v: model.getConstrs()) {
			s +=("Slack: " + v.get(GRB.StringAttr.ConstrName) + " = " + v.get(GRB.DoubleAttr.Slack)) + "\n";
		}
				
		return s;
	}
	
	//returns string for the second request
	private static String secondString (GRBModel model) throws GRBException
	{
		String s = "";
		double obj = model.get(GRB.DoubleAttr.ObjVal); 
		s += ("Obj rilassato: " + obj + "\n");
		s += ("ELENCO NOMI VINCOLI ATTIVI:\n");
		for(GRBConstr c: model.getConstrs()) {
			if (c.get(GRB.DoubleAttr.Slack) >= 0) {
				if (c.get(GRB.DoubleAttr.Slack) < 1e-6) 
					s += c.get(GRB.StringAttr.ConstrName) + "\n";
			}
			else {
				if (c.get(GRB.DoubleAttr.Slack) > -(1e-6)) 
					s += c.get(GRB.StringAttr.ConstrName) + "\n";

			}
		}
		
		return s;
	}

	/*
	 * QUESITO II
	 */
	//do not study a subject for more than 2 consecutive days
	private static void addConstr5 (GRBModel model, GRBVar[][] y) throws GRBException
	{
		for (int i = 0; i < t.size(); i++) {
			for (int j = 2; j < d; j++) {
				GRBLinExpr lhs = new GRBLinExpr();
				lhs.addTerm(1, y[i][j-2]); //day before yesterday
				lhs.addTerm(1, y[i][j-1]); //yesterday 
				lhs.addTerm(1, y[i][j]);   //today
				model.addConstr(lhs, GRB.LESS_EQUAL, l, "quesito2_vincolo2cons_y_" + i + "_" + j);
			}
		}
	}
	//study a if yesterday you did not study b AND c
	private static void addConstr6 (GRBModel model, GRBVar[][] y) throws GRBException
	{
		for (int j = 1; j < d; j++) {
			GRBLinExpr lhs1 = new GRBLinExpr();
			lhs1.addTerm(1, y[a][j]);
			lhs1.addTerm(1, y[b][j-1]); //j-1 = yesterday
			model.addConstr(lhs1, GRB.LESS_EQUAL, 1, "quesito2_studioab_senonstudiatobc_giorno_" + j);
			GRBLinExpr lhs2 = new GRBLinExpr();
			lhs2.addTerm(1, y[a][j]);
			lhs2.addTerm(1, y[c][j-1]);
			model.addConstr(lhs2, GRB.LESS_EQUAL, 1, "quesito2_studioac_senonstudiatobc_giorno_" + j);
			//if you want to force ya=1 if the other variables equal to 0 then you can add another constraint
			// ya(j) + [ yb(j-1) + yc(j-1) ] >= 1
		}
		
	}
	
	/*
	 * QUESITO III
	 */
	//first request, sensitivity analysis of the relaxed model
	private static void sensitivitaRelaxed(GRBModel rilassato) throws GRBException
	{
		/*
		 * Calculate delta
		 * I want the lower bound to be greater than or equal to 0, because we are talking about a constraint that 
		 * limits a time variable. I know that if I go below 0 my constraint is always true, but it's not useful 
		 * for our problem.
		 */
		for (int j = 0; j < d; j++) {
			double pi2 = rilassato.getConstrByName("min_ore_materia_2_giorno_" + j).get(GRB.DoubleAttr.Pi);
			double low = rilassato.getConstrByName("min_ore_materia_2_giorno_" + j).get(GRB.DoubleAttr.SARHSLow);
			double up =  rilassato.getConstrByName("min_ore_materia_2_giorno_" + j).get(GRB.DoubleAttr.SARHSUp);
			soluzioneAVideo += "Intervallo DELTA pi_2 giorno " + j + ": ";
			double min_delta = 0.0;
			if (low >= 0) min_delta = pi2 - low;
			else min_delta = pi2;
			double max_delta = up - pi2;
			soluzioneAVideo += String.format("[%.1f , %.1f]\n", min_delta, max_delta);
		}
	}
	//minimize minval, change max hours a day constraint to sum(vars) <= tmax - minval
	private static void minTmax (GRBEnv env, GRBModel model, GRBVar[][] x)throws GRBException {
		GRBVar minval = model.addVar(0, tmax, 0, GRB.INTEGER, "minval");
		GRBLinExpr fo = new GRBLinExpr();
		fo.addTerm(1, minval);
		model.setObjective(fo);
		
		for (int j = 0; j < d; j++) {
			model.remove(model.getConstrByName("max_ore_giorno_" + j));
			GRBLinExpr exp = new GRBLinExpr();
			for (int i = 0; i < t.size(); i++) {
				exp.addTerm(1, x[i][j]);
			}
			GRBLinExpr rExp = new GRBLinExpr();
			rExp.addConstant(tmax);
			rExp.addTerm(-1, minval);
			model.addConstr(exp, GRB.LESS_EQUAL, rExp, "ore_massime_giorno_" + j);
		}
		model.optimize();
		
		soluzioneAVideo += "valore min tmax: " + (tmax-(int)model.get(GRB.DoubleAttr.ObjVal));
	}
	
	private static void dispose(GRBModel model, GRBModel rilassato, GRBEnv env)
	{
		try
		{
			rilassato.dispose();
			model.dispose();
			env.dispose();
			model = null;
			rilassato = null;
			env = null;
		} catch (GRBException e)
		{
			e.printStackTrace();
		}
	}

}
